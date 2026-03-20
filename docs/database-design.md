# Database Design – IronFlow (Gym Workout Planner)

## 1. Overview

This document defines the database schema for the IronFlow system.

* Database: PostgreSQL
* Design type: Relational Database
* Goal: Ensure data consistency, scalability, and efficient querying

---

## 2. Entity Relationship Overview

Main entities:

* users
* workouts
* exercises
* workout_exercises
* progress_logs

Relationships:

* One user → many workouts
* One workout → many workout_exercises
* One exercise → many workout_exercises
* One user → many progress_logs

---

## 3. Tables Definition

### 3.1 users

Stores user account information.

| Column     | Type      | Constraints      |
| ---------- | --------- | ---------------- |
| id         | UUID      | PK               |
| email      | VARCHAR   | UNIQUE, NOT NULL |
| password   | VARCHAR   | NOT NULL         |
| created_at | TIMESTAMP | DEFAULT NOW()    |

---

### 3.2 workouts

Stores workout schedules.

| Column     | Type      | Constraints   |
| ---------- | --------- | ------------- |
| id         | UUID      | PK            |
| user_id    | UUID      | FK → users.id |
| date       | DATE      | NOT NULL      |
| created_at | TIMESTAMP | DEFAULT NOW() |

---

### 3.3 exercises

Stores available exercises.

| Column       | Type    | Constraints |
| ------------ | ------- | ----------- |
| id           | UUID    | PK          |
| name         | VARCHAR | NOT NULL    |
| muscle_group | VARCHAR | NOT NULL    |

---

### 3.4 workout_exercises

Join table between workouts and exercises.

| Column      | Type | Constraints       |
| ----------- | ---- | ----------------- |
| id          | UUID | PK                |
| workout_id  | UUID | FK → workouts.id  |
| exercise_id | UUID | FK → exercises.id |
| sets        | INT  | NOT NULL          |
| reps        | INT  | NOT NULL          |

---

### 3.5 progress_logs

Stores user workout results.

| Column      | Type  | Constraints       |
| ----------- | ----- | ----------------- |
| id          | UUID  | PK                |
| user_id     | UUID  | FK → users.id     |
| exercise_id | UUID  | FK → exercises.id |
| weight      | FLOAT | NOT NULL          |
| reps        | INT   | NOT NULL          |
| date        | DATE  | NOT NULL          |

---

## 4. Relationships

### 4.1 users → workouts

* One-to-Many
* A user can have multiple workouts

---

### 4.2 workouts → workout_exercises

* One-to-Many
* A workout can contain multiple exercises

---

### 4.3 exercises → workout_exercises

* One-to-Many
* One exercise can appear in multiple workouts

---

### 4.4 users → progress_logs

* One-to-Many

---

### 4.5 exercises → progress_logs

* One-to-Many

---

## 5. Indexing Strategy

To improve performance:

* Index on users.email (UNIQUE)
* Index on workouts.user_id
* Index on workout_exercises.workout_id
* Index on progress_logs.user_id
* Index on progress_logs.exercise_id

---

## 6. Sample SQL Schema

```sql id="m2t9y6"
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  password VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE workouts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE exercises (
  id UUID PRIMARY KEY,
  name VARCHAR NOT NULL,
  muscle_group VARCHAR NOT NULL
);

CREATE TABLE workout_exercises (
  id UUID PRIMARY KEY,
  workout_id UUID REFERENCES workouts(id),
  exercise_id UUID REFERENCES exercises(id),
  sets INT NOT NULL,
  reps INT NOT NULL
);

CREATE TABLE progress_logs (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  exercise_id UUID REFERENCES exercises(id),
  weight FLOAT NOT NULL,
  reps INT NOT NULL,
  date DATE NOT NULL
);
```

---

## 7. Data Integrity Rules

* Foreign keys must always reference existing records
* No orphan records allowed
* Deleting a user should cascade or restrict related data

---

## 8. Scalability Considerations

* Use UUID for distributed systems
* Normalize data to reduce redundancy
* Can extend with:

  * workout_templates
  * user_profiles
  * analytics tables

---

## 9. Future Improvements

* Add `sets_log` table (track each set individually)
* Add `muscle_groups` table (normalize data)
* Add `goals` table for user targets

---

## Conclusion

This database design provides a solid foundation for the IronFlow system.
It ensures clear relationships, scalability, and efficient data management for workout tracking.
