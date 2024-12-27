Django REST Framework (DRF) Spectacular makes it easy to integrate Swagger UI for your API documentation. Here's a step-by-step guide to set it up:

1. Install DRF Spectacular
If you haven't already installed DRF Spectacular, do so with pip:

<!-- //pip install drf-spectacular -->


2. Configure DRF Spectacular in Your Django Settings
Add drf_spectacular to your INSTALLED_APPS and configure it in your settings.py:

<!-- INSTALLED_APPS = [
    ...
    'rest_framework',
    'drf_spectacular',
]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
} -->



3. Create the Schema View
Add a view to generate the OpenAPI schema. This schema will be used by Swagger UI.

In your urls.py:


<!-- from django.urls import path
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView, SpectacularRedocView

urlpatterns = [
    # OpenAPI schema
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),

    # Swagger UI
    path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),

    # Redoc
    path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
] -->


4. Access the Swagger UI
Run your Django server and navigate to:

Swagger UI: http://127.0.0.1:8000/api/schema/swagger-ui/
OpenAPI Schema (JSON): http://127.0.0.1:8000/api/schema/
Redoc: http://127.0.0.1:8000/api/schema/redoc/


5. Customize the Schema (Optional)
You can customize the generated schema using settings in settings.py. For example:


<!-- SPECTACULAR_SETTINGS = {
    'TITLE': 'My API',
    'DESCRIPTION': 'Description of my API',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
} -->

This will adjust the title, description, and other metadata in your Swagger UI.


6. Add Schema Annotations
To improve the documentation, use DRF Spectacular decorators like @extend_schema on your views or viewsets.

Example:
<!-- from drf_spectacular.utils import extend_schema
from rest_framework.views import APIView
from rest_framework.response import Response

class MyView(APIView):
    @extend_schema(
        summary="Example endpoint",
        description="This is an example endpoint description",
    )
    def get(self, request):
        return Response({"message": "Hello, world!"})
 -->

 With these steps, you'll have Swagger UI integrated and functional with Django DRF Spectacular!