# 🛡️ PrivacyShield Lens

A Django-based **facial privacy leak detection system** that scans simulated third-party apps (social media, dating apps, news portals) to detect if a user's photos have been leaked or misused — and allows authorized officers to blur the identified faces.

---

## 📋 Table of Contents

- [How It Works](#how-it-works)
- [Features](#features)
- [User Roles](#user-roles)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Database Models](#database-models)
- [Setup & Installation](#setup--installation)
- [Running the Project](#running-the-project)
- [Face Recognition Engine](#face-recognition-engine)

---

## How It Works

```
User uploads photo → Scan triggered → DeepFace compares photo against fake app images
       ↓                                          ↓
  Stored in DB                          Match found (leak) → Saved to DB
                                                 ↓
                               Admin/Officer reviews → Blurs face in leaked image
```

1. **User registers** and uploads a personal photo (e.g., their face photo).
2. The system **scans simulated "fake apps"** (social media, dating app, news portal) by comparing the uploaded photo against every image displayed in those apps.
3. If a face match is found using **AI-powered facial recognition (DeepFace + Facenet512)**, it is recorded as a **leak**.
4. The user can view their **scan results and confidence scores**.
5. **Admin or Officer** can review leaks and trigger a **face blur/masking** action to protect the user's privacy in the leaked image.
6. Users can also **file complaints**, which officers manage and resolve.

---

## Features

### 👤 User Features
| Feature | Description |
|---|---|
| **Register / Login** | Email-based authentication (no username) |
| **Upload Photo** | Upload a personal photo to monitor |
| **Scan for Leaks** | AI scan against all fake app images |
| **View Results** | See where your photo was leaked with confidence % |
| **File Complaints** | Report privacy violations |
| **View History** | Track all past scans and results |

### 🔐 Admin Features
| Feature | Description |
|---|---|
| **Admin Dashboard** | View all users, officers, and leak statistics |
| **Add Officers** | Create officer accounts assigned to districts |
| **Run Scans** | Trigger scans on behalf of any user |
| **Mask Leaked Images** | Blur user's face in leaked images using OpenCV |
| **Manage Complaints** | Assign complaints to officers, update status |
| **Global Leak Report** | View all leaks across the system |

### 👮 Officer Features
| Feature | Description |
|---|---|
| **District-based Access** | Officers only see users from their district |
| **Run Scans** | Scan images for assigned users |
| **Manage Complaints** | Handle complaints in their jurisdiction |
| **Face Masking** | Blur faces in leaked images |

### 🎭 Fake Demo Apps (Simulated Leak Sources)
| App | Path | Purpose |
|---|---|---|
| **Social Media App** | `/fake-app/social-media/` | Simulates a Facebook-like feed with user images |
| **Dating App** | `/fake-app/dating-app/` | Simulates a Tinder-like matching app |
| **News Portal** | `/fake-app/news-portal/` | Simulates a news site with suspect images |

---

## User Roles

```
┌─────────────────────────────────────────────────────────┐
│                        ADMIN                            │
│  Full access: all users, officers, scans, complaints    │
└───────────────────────┬─────────────────────────────────┘
                        │ manages
         ┌──────────────▼──────────────┐
         │          OFFICER             │
         │  District-scoped: users in   │
         │  their assigned district     │
         └──────────────┬──────────────┘
                        │ handles
         ┌──────────────▼──────────────┐
         │            USER              │
         │  Upload photos, run scans,   │
         │  view results, complaints    │
         └─────────────────────────────┘
```

Officers are assigned to Kerala **districts** (e.g., Ernakulam, Thrissur). They can only view and manage users from their own district.

---

## Technology Stack

### Backend
| Technology | Version | Purpose |
|---|---|---|
| **Python** | 3.11.2 | Core language |
| **Django** | 3.2.7 | Web framework, ORM, routing, auth |
| **MySQL** | — | Relational database |
| **mysqlclient** | 2.2.8 | Django ↔ MySQL connector |

### AI / Machine Learning
| Technology | Version | Purpose |
|---|---|---|
| **DeepFace** | 0.0.99 | High-level face recognition API |
| **Facenet512** | — | Face recognition model (90.6 MB weights) |
| **VGGFace** | — | Backup model (553 MB weights) |
| **TensorFlow** | 2.20.0 | Deep learning backend for DeepFace |
| **Keras** | 3.12.1 | Neural network layer for TF |
| **OpenCV** | 4.13.0 | Face detection (Haar Cascade) for masking |
| **NumPy** | 2.2.6 | Array operations for image processing |
| **Pillow** | 12.1.1 | Image loading, cropping, saving |
| **SciPy** | 1.15.3 | Distance metric calculations |

### Frontend
| Technology | Purpose |
|---|---|
| **HTML5 / CSS3** | Page structure and styling |
| **Bootstrap 5.3** | Responsive layout, components |
| **Bootstrap Icons** | UI icon set |
| **Font Awesome 6** | Additional icons (dating app) |
| **Google Fonts (Inter)** | Modern typography |
| **Django Templates** | Server-side HTML rendering |
| **Vanilla JavaScript** | Form validation, interactive UI |

### Other Libraries
| Library | Purpose |
|---|---|
| **python-dotenv** | Environment variable management |
| **gunicorn** | Production WSGI server |
| **retina-face** | Face detection backend for DeepFace |
| **mtcnn** | Alternative face detector |
| **pandas** | Data processing |
| **cryptography** | Secure operations |

---

## Project Structure

```
backend/
│
├── backend/                    # Django project config
│   ├── settings.py             # App settings, DB, media config
│   ├── urls.py                 # Root URL router
│   └── wsgi.py
│
├── login/                      # Authentication app
│   ├── models.py               # Custom User model (email-based, role-based)
│   ├── views.py                # register_view, login_view, logout_view
│   └── urls.py
│
├── leak_detection/             # Core privacy detection app
│   ├── models.py               # UploadedImage, LeakScanResult, Complaint
│   ├── views.py                # All user, admin, officer, fake-app views
│   ├── utils_face_rec.py       # 🧠 AI face recognition engine
│   └── urls.py                 # All routes
│
├── templates/
│   ├── login/
│   │   ├── login.html
│   │   └── register.html       # Kerala district dropdown
│   ├── leak_detection/
│   │   ├── base.html           # Shared navbar/layout
│   │   ├── user_dashboard.html
│   │   ├── upload_image.html
│   │   ├── check_leaks.html
│   │   ├── admin_dashboard.html
│   │   ├── admin_leak_detection.html
│   │   ├── complaint_list.html
│   │   └── ...
│   └── fake_apps/
│       ├── social_media.html   # Fake social feed
│       ├── dating_app.html     # Fake dating cards
│       └── news_portal.html    # Fake news site
│
├── media/
│   ├── user_images/            # Uploaded user photos
│   ├── profile/                # User profile pictures
│   └── fake_apps/
│       ├── social_media/       # alan.jpeg, alice_profile.jpg
│       ├── dating_app/         # alan2.jpg, fake_profile.jpg
│       └── news_portal/        # suspect.jpg, ...
│
├── static/
│   └── css/                    # Custom stylesheets
│
└── requirements.txt            # All Python dependencies
```

---

## Database Models

### `User` (login app)
| Field | Type | Description |
|---|---|---|
| `email` | EmailField (unique) | Login identifier (no username) |
| `fullname` | CharField | Full name |
| `phone` | CharField | Phone number |
| `address` | TextField | Home address |
| `age` | IntegerField | Age |
| `place` | CharField | Kerala district |
| `profile` | ImageField | Profile photo |
| `role` | CharField | `user` / `officer` / `admin` |
| `status` | CharField | `active` / `blocked` |
| `assigned_officer` | ForeignKey(self) | Officer responsible for this user |

### `UploadedImage` (leak_detection app)
| Field | Type | Description |
|---|---|---|
| `user` | ForeignKey(User) | Owner of the image |
| `image` | ImageField | Stored in `media/user_images/` |
| `uploaded_at` | DateTimeField | Upload timestamp |

### `LeakScanResult` (leak_detection app)
| Field | Type | Description |
|---|---|---|
| `user` | ForeignKey(User) | User whose image was scanned |
| `target_image` | ForeignKey(UploadedImage) | The scanned source photo |
| `source_app_name` | CharField | e.g., "Dating App", "Social Media App" |
| `matched_image_path` | CharField | Web URL of the leaked image |
| `confidence_score` | FloatField | Match confidence (0–100%) |
| `is_leaked` | BooleanField | True if a face match was confirmed |
| `scan_date` | DateTimeField | When the scan ran |

### `Complaint` (leak_detection app)
| Field | Type | Description |
|---|---|---|
| `user` | ForeignKey(User) | Who filed the complaint |
| `subject` | CharField | Complaint title |
| `description` | TextField | Detailed description |
| `status` | CharField | `pending` / `investigating` / `resolved` / `closed` |
| `assigned_officer` | ForeignKey(User) | Officer handling it |
| `created_at` | DateTimeField | Filing timestamp |

---

## Setup & Installation

### Prerequisites
- Python 3.11.x
- MySQL Server (with a database named `privacy_protection`)
- Git

### 1. Clone & Navigate
```bash
git clone <repo-url>
cd "privacy protection/backend"
```

### 2. Create & Activate Virtual Environment
```bash
python -m venv env

# Windows
env\Scripts\activate

# macOS/Linux
source env/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure Database
Edit `backend/settings.py` with your MySQL credentials:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'privacy_protection',
        'USER': 'root',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

### 5. Run Migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### 6. Create Admin Superuser
```bash
python manage.py createsuperuser
```
> Enter your email and password. The admin can log in at `/admin-dashboard/`.

### 7. Download AI Model Weights
The face recognition models are downloaded automatically on first scan. If auto-download fails (firewall/network issues), download manually:

```bash
# Create the weights directory
mkdir %USERPROFILE%\.deepface\weights

# Download Facenet512 (~91 MB) — primary model
curl -L -o %USERPROFILE%\.deepface\weights\facenet512_weights.h5 ^
  https://github.com/serengil/deepface_models/releases/download/v1.0/facenet512_weights.h5

# Download VGGFace (~553 MB) — backup model  
curl -L -o %USERPROFILE%\.deepface\weights\vgg_face_weights.h5 ^
  https://github.com/serengil/deepface_models/releases/download/v1.0/vgg_face_weights.h5
```

### 8. Add Fake App Images
Place test images in the `media/fake_apps/` directories:
```
media/
└── fake_apps/
    ├── social_media/    ← images shown in social_media.html
    ├── dating_app/      ← images shown in dating_app.html
    └── news_portal/     ← images shown in news_portal.html
```
The scan engine automatically reads which images are referenced in the HTML templates — no manual configuration needed.

---

## Running the Project

```bash
# Activate virtual environment first
env\Scripts\activate

# Start the development server
python manage.py runserver
```

Open your browser at: **http://127.0.0.1:8000/**

| URL | Page |
|---|---|
| `/` | Redirects to login |
| `/register/` | Create a new account |
| `/login/` | Sign in |
| `/dashboard/` | User dashboard (uploaded images) |
| `/upload/` | Upload a photo |
| `/check-leaks/` | View scan results |
| `/admin-dashboard/` | Admin panel |
| `/fake-app/social-media/` | Demo: Fake social media |
| `/fake-app/dating-app/` | Demo: Fake dating app |
| `/fake-app/news-portal/` | Demo: Fake news portal |

---

## Face Recognition Engine

The core scanning logic lives in [`leak_detection/utils_face_rec.py`](leak_detection/utils_face_rec.py).

### How the Scan Works

```python
# Step 1: Parse all fake app HTML templates
# Find every image path referenced like: {{ MEDIA_URL }}fake_apps/dating_app/alan2.jpg
shown_images = get_images_shown_in_templates()

# Step 2: For each image found in templates
for image_path in shown_images:
    result = DeepFace.verify(
        img1_path=user_uploaded_photo,
        img2_path=image_path,
        model_name="Facenet512",   # 512-dimensional face embedding
        enforce_detection=False    # Don't fail if face not perfectly detected
    )
    # Step 3: Custom threshold (0.40) — more lenient than default 0.30
    # Catches same person in different lighting/angles
    is_match = result['distance'] < 0.40
```

### Threshold Tuning
| Distance | Meaning |
|---|---|
| `< 0.10` | Near-identical photos |
| `0.10 – 0.30` | Same person, different conditions |
| `0.30 – 0.40` | Likely same person (our match zone) |
| `> 0.40` | Different person |

### Face Masking (Blur)
When an admin/officer triggers masking:
1. **OpenCV Haar Cascade** detects all faces in the leaked image
2. **DeepFace** verifies each detected face against the user's reference photo
3. **Only the matching face** is blurred using Gaussian blur (not other faces in the same image)
4. The original image is **backed up** before modification

---

## Notes

- The project is designed for **educational/demonstration purposes** to show how facial privacy leaks can be detected and mitigated.
- The "fake apps" are internal Django views that simulate third-party platforms — in a real-world scenario, these would be external websites crawled via scraping.
- All user data and uploaded images are stored locally; no external services are used for image storage.
