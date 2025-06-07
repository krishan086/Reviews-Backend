Overview: Gig Review System – Flow & Architecture
Tech Stack:
Backend: Django, Django REST Framework (DRF)

Database: SQLite (or any DB used)

Authentication: Token Authentication

API Tool: Postman for CRUD testing

```
System Flow:-
                ┌─────────────┐
                │   Client    │
                │(Postman/UI) │
                └────┬────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │  Send HTTP Request     │
        │(GET, POST, PUT, DELETE)│
        └─────────┬──────────────┘
                  ▼
       ┌──────────────────────┐
       │ Django REST Framework│
       │   ViewSet Layer      │
       └────────┬─────────────┘
                ▼
      ┌────────────────────────┐
      │   ReviewSerializer     │
      │  (Validate, format)    │
      └────────┬───────────────┘
               ▼
     ┌────────────────────────────┐
     │        Review Model        │
     │(user FK, gig FK, rating...)│
     └────────┬───────────────────┘
              ▼
     ┌────────────────────────────┐
     │        Database (SQLite)   │
     └────────────────────────────┘

```

Review Model:
class Review(models.Model):
    gig = models.ForeignKey(Gig, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    rating = models.DecimalField(...)
    comment = models.TextField(...)

views:
class ReviewViewSet(viewsets.ModelViewSet):
    queryset = Review.objects.all()
    serializer_class = ReviewSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

serializers:
class ReviewSerializer(serializers.ModelSerializer):
    user_id = serializers.IntegerField(source='user.id', read_only=True)
    gig_id = serializers.IntegerField()

    def create(self, validated_data):
        gig = Gig.objects.get(id=validated_data.pop('gig_id'))
        return Review.objects.create(user=self.context['request'].user, gig=gig, **validated_data)


Makes /api/reviews/ the main endpoint.

Supports:

GET /api/reviews/

POST /api/reviews/

PUT /api/reviews/{id}/

DELETE /api/reviews/{id}/


Flowchart of File Structure:

```
gigportal/                  <-- Django project root
│
├── gigportal/              <-- Project settings module
│   ├── __init__.py
│   ├── settings.py         <-- Main settings file
│   ├── urls.py             <-- Project-level URLs (includes app URLs)
│   ├── asgi.py
│   └── wsgi.py
│
├── reviews/                <-- Your main app for reviews
│   ├── __init__.py
│   ├── admin.py            <-- Admin site config
│   ├── apps.py             <-- App config
│   ├── migrations/         <-- DB migration files
│   │   └── __init__.py
│   ├── models.py           <-- Review & Gig models
│   ├── serializers.py      <-- DRF serializers
│   ├── tests.py            <-- Unit tests (optional)
│   ├── urls.py             <-- App-specific URLs (router for ReviewViewSet)
│   └── views.py            <-- API viewsets (ReviewViewSet)
│
├── manage.py               <-- Django management CLI
│
└── requirements.txt        <-- (Optional) Python dependencies list
```

