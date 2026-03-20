# Planning – IronFlow (Gym Workout Planner)

## 1. Feature Scope

### 1.1 MVP (v1)

Core features for the first release:

* Create workout schedules (by date)
* Add exercises to a workout
* Define sets and reps for each exercise
* Log workout results (weight, reps)
* View basic workout history

---

### 1.2 Future Versions

#### v2

* Progress tracking (charts, stats)
* Workout reminders (notifications)
* Predefined workout templates

#### v3

* AI-based workout suggestions
* Smart recommendations based on user history

---

## 2. User Flow

### 2.1 Main Flow

User journey from start to finish:

1. User opens the app
2. User creates a workout schedule
3. User adds exercises to the workout
4. User performs workout
5. User logs results (reps, weight)
6. User reviews progress

---

### 2.2 Authentication Flow

1. User registers account
2. User logs in
3. System authenticates user
4. User accesses dashboard

---

### 2.3 Workout Creation Flow

1. User selects a date
2. User creates a new workout
3. User selects exercises
4. User sets:

   * Number of sets
   * Number of reps
5. User saves workout

---

### 2.4 Workout Tracking Flow

1. User opens today's workout
2. User inputs:

   * Weight
   * Reps completed
3. User submits log
4. System saves progress

---

## 3. System Modules

### 3.1 Authentication Module

* Register
* Login
* JWT-based authentication

---

### 3.2 Workout Module

* Create workout
* Update workout
* Delete workout
* View workouts by date

---

### 3.3 Exercise Module

* List available exercises
* Categorize by muscle group

---

### 3.4 Progress Module

* Log workout results
* Retrieve progress history

---

## 4. Data Flow Overview

1. User interacts with frontend
2. Frontend sends request to backend API
3. Backend processes logic
4. Backend interacts with database
5. Response returned to frontend
6. UI updates accordingly

---

## 5. Non-Functional Requirements

### Performance

* Fast response time (< 300ms for API)

### Usability

* Clean and simple UI
* Easy to use for beginners

### Scalability

* Modular backend structure
* Ready for feature expansion

### Security

* Password hashing
* JWT authentication

---

## 6. Constraints

* Limited development time (solo project)
* Focus on core features only (avoid feature overload)
* Prioritize simplicity over complexity

---

## 7. Success Criteria

The project is considered successful if:

* Users can create and manage workout schedules
* Users can log workout results
* The system is stable and easy to use
* Codebase is clean and maintainable

---

## 8. Development Strategy

* Build MVP first
* Keep UI minimal but functional
* Write clean and modular code
* Document clearly for portfolio purposes

---

## 9. Future Expansion Ideas

* Mobile app (React Native)
* Social features (share workouts)
* Integration with wearable devices
* Advanced analytics dashboard

---

## Conclusion

This planning document defines the structure and direction of the IronFlow project.
The focus is on delivering a simple, effective, and scalable workout planning system.
