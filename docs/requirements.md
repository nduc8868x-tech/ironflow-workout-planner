# Requirements Document – IronFlow (Gym Workout Planner)

## 1. Overview

This document defines the functional and non-functional requirements for the IronFlow system, including frontend, backend, and database specifications.

---

## 2. System Architecture

* Frontend: Next.js (Client-side UI)
* Backend: Node.js (REST API)
* Database: PostgreSQL

Communication flow:
Frontend → Backend API → Database

---

## 3. Functional Requirements

### 3.1 Authentication

#### Features

* User registration
* User login
* Secure authentication using JWT

#### API

**POST /auth/register**

* Input:

  * email (string, required)
  * password (string, required, min 6 chars)
* Output:

  * user_id
  * token

---

**POST /auth/login**

* Input:

  * email
  * password
* Output:

  * token

---

### 3.2 Workout Management

#### Features

* Create workout
* Update workout
* Delete workout
* Get workouts by user

#### API

**GET /workouts**

* Headers:

  * Authorization: Bearer token
* Output:

  * List of workouts

---

**POST /workouts**

* Input:

  * date (date)
* Output:

  * workout_id

---

**PUT /workouts/:id**

* Input:

  * date
* Output:

  * updated workout

---

**DELETE /workouts/:id**

* Output:

  * success message

---

### 3.3 Exercise Management

#### Features

* Retrieve exercise list
* Categorize exercises by muscle group

#### API

**GET /exercises**

* Output:

  * id
  * name
  * muscle_group

---

### 3.4 Workout Exercise

#### Features

* Add exercises to a workout
* Define sets and reps

#### API

**POST /workout-exercises**

* Input:

  * workout_id
  * exercise_id
  * sets (int)
  * reps (int)

---

**DELETE /workout-exercises/:id**

---

### 3.5 Progress Tracking

#### Features

* Log workout results
* View progress history

#### API

**POST /progress**

* Input:

  * exercise_id
  * weight (float)
  * reps (int)
  * date

---

**GET /progress**

* Output:

  * list of logs

---

## 4. Frontend Requirements

### Pages

* Dashboard
* Workout Calendar
* Workout Detail Page
* Progress Page
* Authentication صفحات (Login/Register)

---

### UI Features

* Display workouts by date
* Form to create/edit workouts
* Input fields for reps and weight
* Simple and clean layout

---

## 5. Database Requirements

### Tables

#### users

* id (PK)
* email (unique)
* password

---

#### workouts

* id (PK)
* user_id (FK)
* date

---

#### exercises

* id (PK)
* name
* muscle_group

---

#### workout_exercises

* id (PK)
* workout_id (FK)
* exercise_id (FK)
* sets
* reps

---

#### progress_logs

* id (PK)
* user_id (FK)
* exercise_id (FK)
* weight
* reps
* date

---

## 6. Validation Rules

### Authentication

* Email must be valid format
* Password minimum 6 characters

### Workout

* Date is required

### Exercise

* sets > 0
* reps > 0

### Progress

* weight ≥ 0
* reps > 0

---

## 7. Non-Functional Requirements

### Performance

* API response time < 300ms

### Security

* Password hashing (bcrypt)
* JWT authentication

### Usability

* Simple UI
* Easy navigation

### Maintainability

* Modular code structure
* Clear naming conventions

---

## 8. Error Handling

* Return proper HTTP status codes:

  * 200 OK
  * 400 Bad Request
  * 401 Unauthorized
  * 404 Not Found
  * 500 Internal Server Error

---

## 9. Assumptions

* Users have basic knowledge of gym exercises
* Internet connection is stable
* System is used by individual users (not enterprise scale)

---

## 10. Future Enhancements

* Role-based access (trainer / user)
* Analytics dashboard
* AI workout recommendation
* Mobile application support

---

## Conclusion

This document defines the core requirements for building the IronFlow system.
It ensures clarity in development and provides a strong foundation for scalable implementation.
