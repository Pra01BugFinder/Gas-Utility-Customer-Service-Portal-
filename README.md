# Gas-Utility-Customer-Service-Portal-
_Features:_ - Online service request submission with file attachment - Request tracking with status updates - Account information viewing - Customer support tool for managing requests

I provided a simple Django application code earlier that meets the requirements. Here's a breakdown of the code with explanations:

Models
The models define the structure of the data stored in the database.

```
models.py
from django.db import models
from django.contrib.auth.models import User

class ServiceRequest(models.Model):
    REQUEST_TYPES = [
        ('gas_leak', 'Gas Leak'),
        ('meter_issue', 'Meter Issue'),
        ('other', 'Other'),
    ]

    customer = models.ForeignKey(User, on_delete=models.CASCADE)
    request_type = models.CharField(max_length=20, choices=REQUEST_TYPES)
    details = models.TextField()
    attachment = models.FileField(upload_to='attachments/')
    status = models.CharField(max_length=20, choices=[
        ('pending', 'Pending'),
        ('in_progress', 'In Progress'),
        ('resolved', 'Resolved'),
    ], default='pending')
    submitted_at = models.DateTimeField(auto_now_add=True)
    resolved_at = models.DateTimeField(null=True, blank=True)

class RequestUpdate(models.Model):
    request = models.ForeignKey(ServiceRequest, on_delete=models.CASCADE)
    update = models.TextField()
    updated_at = models.DateTimeField(auto_now_add=True)
```

Views
The views handle HTTP requests and return HTTP responses.

```
views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from .models import ServiceRequest, RequestUpdate
from .forms import ServiceRequestForm, RequestUpdateForm

@login_required
def submit_request(request):
    # Handle submitting a new service request
    if request.method == 'POST':
        form = ServiceRequestForm(request.POST, request.FILES)
        if form.is_valid():
            service_request = form.save(commit=False)
            service_request.customer = request.user
            service_request.save()
            return redirect('request_list')
    else:
        form = ServiceRequestForm()
    return render(request, 'submit_request.html', {'form': form})

@login_required
def request_list(request):
    # Display a list of service requests for the current user
    requests = ServiceRequest.objects.filter(customer=request.user)
    return render(request, 'request_list.html', {'requests': requests})

@login_required
def request_detail(request, pk):
    # Display details of a specific service request
    service_request = ServiceRequest.objects.get(pk=pk)
    updates = RequestUpdate.objects.filter(request=service_request)
    return render(request, 'request_detail.html', {'request': service_request, 'updates': updates})
```

Forms
The forms handle user input validation.

```
forms.py
from django import forms
from .models import ServiceRequest, RequestUpdate

class ServiceRequestForm(forms.ModelForm):
    class Meta:
        model = ServiceRequest
        fields = ('request_type', 'details', 'attachment')

class RequestUpdateForm(forms.ModelForm):
    class Meta:
        model = RequestUpdate
        fields = ('update',)
```

Templates
The templates define the HTML structure of the pages.

```
<!-- submit_request.html -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>

<!-- request_list.html -->
<ul>
    {% for request in requests %}
        <li>
            {{ request.request_type }} - {{ request.status }}
            <a href="{% url 'request_detail' request.pk %}">View Details</a>
        </li>
    {% endfor %}
</ul>

<!-- request_detail.html -->
<h1>Request Details</h1>
<p>Request Type: {{ request.request_type }}</p>
<p>Status: {{ request.status }}</p>
<p>Details: {{ request.details }}</p>
<p>Attachment: {{ request.attachment }}</p>
<h2>Updates</h2>
<ul>
    {% for update in updates %}
        <li>{{ update.update }} - {{ update.updated_at }}</li>
    {% endfor %}
</ul>
```

URLs
The URLs define the mapping between URLs and views.

```
urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('submit_request/', views.submit_request, name='submit_request'),
    path('request_list/', views.request_list, name='request_list'),
    path('request_detail/<pk>/', views.request_detail, name='request_detail'),
]
```

This code provides a basic structure for submitting and tracking service requests.
