# MyFit

An app that tracks and logs all the workouts you've done.

## Features

These are the core features for the app:
1. Ability to log their workouts for the day
2. Workouts contain - type,  a list of exercises with sets, reps, weights and / or duration
3. Analytics on progression of weights by muscle group and time for cardio workouts
4. Ability to track body weight, fat percentages and body measurements
5. Ability to create an account
6. Ability to create a workout plan

Extended features:
1. Workout plan generation - use LLMs ?
2. Automatic calendar event generation based on plan
3. Integrate with Apple watch, Google fit, Samsung Watch to get list of workouts, calories spent, steps etc.
 

## Tech Stack

This will be both developed as a mobile application and a website. These are the tools and languages to be used for the app:

| Category | Technology                    |
| -------- | ----------------------------- |
| UI       | React Native                  |
| API      | Java Springboot               |
| DB       | Postgres                      |
| Platform | Android, Apple, Web interface |

## Business Model

### Freemium B2C
Free features 
1. Log workouts - strength and cardio
2. Log weight, fat percentages and body measurements
3. Create a custom workout plan

Premium features (subscription) - 
1. All Free features
2. Analytics on weight changes, fat % changes and body measurement changes
3. Personalized workout plan generation using AI
4. Integration with fitness trackers

### B2B Subscription

This business model focuses on selling this app as a service to Gym Companies. Some features include:

1. Gyms can have their personal trainers use this app to train their customers
2. Customers login with gym memberships
3. Trainers can generate customized workout plans for customers
4. Trainers can track the workouts done by customers
5. Trainees can track the calories consumed by tracking the foods they eat. This will be shown in these main values: carbs, protein, fat and total calories.
6. Trainers can track the calories consumed and spent by the customers and can give personalized feedback to them
