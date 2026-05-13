# Study Buddy — Complete Setup Guide
**Stack:** Django + React + PostgreSQL + OpenAI + JWT + Deploy on Render/Vercel
**OS:** Windows
**Author:** Mohammad Salhab
**Date:** 2026
---
## Table of Contents
1. [Prerequisites — Install everything](#phase-1--prerequisites)
2. [Git setup + Remote repo](#phase-2--git-setup)
3. [Backend — Django + DRF + JWT](#phase-3--backend)
4. [Database — PostgreSQL](#phase-4--postgresql)
5. [Frontend — React + Axios + Auth](#phase-5--frontend)
6. [Run locally](#phase-6--run-locally)
7. [Connect OpenAI API](#phase-7--openai-api)
8. [Deploy on Render + Vercel](#phase-8--deployment)
9. [Quick reference commands](#quick-reference)
---
## PHASE 1 — Prerequisites
### 1.1 Install Git
- Download: https://git-scm.com/download/win
- During install: choose **"Git from command line and also from 3rd-party software"**
- Verify:
```bash
git --version
```
### 1.2 Install Python 3.11
- Download: https://www.python.org/downloads/
- CHECK "Add Python to PATH" during install
- Verify:
```bash
python --version
pip --version
```
### 1.3 Install Node.js LTS (20.x)
- Download: https://nodejs.org
- Run installer (all defaults)
- Verify:
```bash
node -v
npm -v
```
### 1.4 Install PostgreSQL 16
- Download: https://www.postgresql.org/download/windows/
- During install:
 - Set postgres superuser password (REMEMBER IT)
 - Port: 5432 (default)
 - Keep pgAdmin 4 checked
- Verify:
```bash
psql --version
```
### 1.5 Install VS Code
- Download: https://code.visualstudio.com
- Install these extensions:
 - Python (Microsoft)
 - ES7+ React/Redux snippets
 - Prettier
 - GitLens (optional)
### 1.6 Verify all tools
```bash
git --version
python --version
pip --version
node -v
npm -v
psql --version
```
---
## PHASE 2 — Git Setup
### 2.1 Create project folder
```bash
cd C:\Users\<YourName>\Desktop
mkdir study-buddy
cd study-buddy
code .
```
### 2.2 Initialize Git
```bash
git init
git branch -M main
```
### 2.3 Create .gitignore
Create file: `study-buddy/.gitignore`
```
# Python
__pycache__/
*.pyc
*.pyo
.env
.venv/
venv/
*.egg-info/
# Django
db.sqlite3
staticfiles/
media/
# Node / React
node_modules/
dist/
build/
.env.local
# VS Code
.vscode/
# OS
.DS_Store
Thumbs.db
```
### 2.4 Create README
```bash
echo # Study Buddy > README.md
```
### 2.5 First commit
```bash
git add .
git commit -m "chore: init project"
```
### 2.6 Create GitHub remote repo
1. Go to github.com → New repository
2. Name: `study-buddy`
3. Do NOT add README/gitignore (already have them)
4. Copy the HTTPS URL
```bash
git remote add origin https://github.com/<YOUR_USERNAME>/study-buddy.git
git push -u origin main
```
---
## PHASE 3 — Backend (Django + DRF + JWT)
### 3.1 Create backend folder + virtual environment
```bash
mkdir backend
cd backend
python -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
```
### 3.2 Install packages
```bash
pip install django djangorestframework django-cors-headers djangorestframeworksimplejwt psycopg2-binary python-dotenv openai gunicorn whitenoise
pip freeze > requirements.txt
```
### 3.3 Start Django project
```bash
django-admin startproject config .
python manage.py startapp api
```
### 3.4 Folder structure (backend/)
```
backend/
 config/
 __init__.py
 settings.py
 urls.py
 wsgi.py
 api/
 migrations/
 __init__.py
 models.py
 views.py
 serializers.py
 urls.py
 .venv/
 manage.py
 requirements.txt
 .env
```
### 3.5 Create backend/.env
```
DJANGO_SECRET_KEY=replace-this-with-a-long-random-string-min-50-chars
DJANGO_DEBUG=True
DB_NAME=studybuddy_db
DB_USER=studybuddy_user
DB_PASSWORD=your_strong_password
DB_HOST=localhost
DB_PORT=5432
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3000
```
### 3.6 settings.py
```python
import os
from pathlib import Path
from dotenv import load_dotenv
from datetime import timedelta
load_dotenv()
BASE_DIR = Path(__file__).resolve().parent.parent
SECRET_KEY = os.getenv("DJANGO_SECRET_KEY", "fallback-dev-key")
DEBUG = os.getenv("DJANGO_DEBUG", "True") == "True"
ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")
INSTALLED_APPS = [
 "django.contrib.admin",
 "django.contrib.auth",
 "django.contrib.contenttypes",
 "django.contrib.sessions",
 "django.contrib.messages",
 "django.contrib.staticfiles",
 "rest_framework",
 "corsheaders",
 "api",
]
MIDDLEWARE = [
 "corsheaders.middleware.CorsMiddleware",
 "django.middleware.security.SecurityMiddleware",
 "whitenoise.middleware.WhiteNoiseMiddleware",
 "django.contrib.sessions.middleware.SessionMiddleware",
 "django.middleware.common.CommonMiddleware",
 "django.middleware.csrf.CsrfViewMiddleware",
 "django.contrib.auth.middleware.AuthenticationMiddleware",
 "django.contrib.messages.middleware.MessageMiddleware",
 "django.middleware.clickjacking.XFrameOptionsMiddleware",
]
ROOT_URLCONF = "config.urls"
TEMPLATES = [
 {
 "BACKEND": "django.template.backends.django.DjangoTemplates",
 "DIRS": [],
 "APP_DIRS": True,
 "OPTIONS": {
 "context_processors": [
 "django.template.context_processors.debug",
 "django.template.context_processors.request",
 "django.contrib.auth.context_processors.auth",
 "django.contrib.messages.context_processors.messages",
 ],
 },
 },
]
WSGI_APPLICATION = "config.wsgi.application"
DATABASES = {
 "default": {
 "ENGINE": "django.db.backends.postgresql",
 "NAME": os.getenv("DB_NAME"),
 "USER": os.getenv("DB_USER"),
 "PASSWORD": os.getenv("DB_PASSWORD"),
 "HOST": os.getenv("DB_HOST", "localhost"),
 "PORT": os.getenv("DB_PORT", "5432"),
 }
}
AUTH_PASSWORD_VALIDATORS = [
 {"NAME": "django.contrib.auth.password_validation.MinimumLengthValidator"},
]
REST_FRAMEWORK = {
 "DEFAULT_AUTHENTICATION_CLASSES": (
 "rest_framework_simplejwt.authentication.JWTAuthentication",
 ),
 "DEFAULT_PERMISSION_CLASSES": (
 "rest_framework.permissions.IsAuthenticated",
 ),
}
SIMPLE_JWT = {
 "ACCESS_TOKEN_LIFETIME": timedelta(minutes=60),
 "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
 "AUTH_HEADER_TYPES": ("Bearer",),
}
CORS_ALLOWED_ORIGINS = os.getenv(
 "CORS_ALLOWED_ORIGINS", "http://localhost:5173"
).split(",")
STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
LANGUAGE_CODE = "en-us"
TIME_ZONE = "Asia/Amman"
USE_I18N = True
USE_TZ = True
```
### 3.7 models.py
```python
from django.db import models
from django.contrib.auth.models import User
class Note(models.Model):
 user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="notes")
 title = models.CharField(max_length=255)
 content = models.TextField()
 created_at = models.DateTimeField(auto_now_add=True)
 updated_at = models.DateTimeField(auto_now=True)
 def __str__(self):
 return f"{self.user.username} — {self.title}"
class BugReport(models.Model):
 PRIORITY = [("low", "Low"), ("medium", "Medium"), ("high", "High")]
 user = models.ForeignKey(
 User, on_delete=models.SET_NULL,
 null=True, blank=True, related_name="reports"
 )
 title = models.CharField(max_length=255)
 description = models.TextField()
 steps = models.TextField()
 expected = models.TextField()
 actual = models.TextField()
 browser = models.CharField(max_length=100, blank=True)
 priority = models.CharField(max_length=10, choices=PRIORITY, default="medium")
 created_at = models.DateTimeField(auto_now_add=True)
 def __str__(self):
 return self.title
```
### 3.8 serializers.py
```python
from django.contrib.auth.models import User
from rest_framework import serializers
from .models import Note, BugReport
class RegisterSerializer(serializers.ModelSerializer):
 password = serializers.CharField(write_only=True, min_length=8)
 class Meta:
 model = User
 fields = ("id", "username", "email", "password")
 def create(self, validated_data):
 return User.objects.create_user(
 username=validated_data["username"],
 email=validated_data.get("email", ""),
 password=validated_data["password"],
 )
class UserSerializer(serializers.ModelSerializer):
 class Meta:
 model = User
 fields = ("id", "username", "email")
class NoteSerializer(serializers.ModelSerializer):
 class Meta:
 model = Note
 fields = ("id", "title", "content", "created_at", "updated_at")
 read_only_fields = ("id", "created_at", "updated_at")
class BugReportSerializer(serializers.ModelSerializer):
 class Meta:
 model = BugReport
 fields = "__all__"
 read_only_fields = ("id", "created_at", "user")
```
### 3.9 views.py
```python
import os
import json
from openai import OpenAI
from django.contrib.auth.models import User
from rest_framework import generics, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework.response import Response
from .models import Note, BugReport
from .serializers import (
 RegisterSerializer, UserSerializer,
 NoteSerializer, BugReportS
