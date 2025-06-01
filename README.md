# FuelFlux - Fuel Station Management System

## Project Overview
FuelFlux is a comprehensive fuel station management system designed to streamline operations, monitor transactions, manage employees, track safety records, and enhance customer experience through online booking and administration.

The project is built with HTML, CSS, and JavaScript, with Firebase integration for backend functionality including authentication, database, storage, and hosting.

## Pages and Features

### 1. Landing Page (index.html)
- **Description**: Main entry point for the application
- **Features**:
  - Modern, responsive design with the FuelFlux branding
  - Call-to-action buttons directing to login/register
  - Mobile-friendly layout

### 2. Login Page (Pages/login.html)
- **Description**: User authentication portal
- **Features**:
  - Login form with email/password authentication
  - Registration option for new users
  - Responsive design with modern UI
  - "Remember me" and "Forgot password" functionality
  - Firebase Authentication integration

### 3. Admin Dashboard (Pages/Admin.html)
- **Description**: Central control panel for administrators
- **Features**:
  - Dashboard overview with key metrics
  - Sidebar navigation to other admin sections
  - Real-time statistics display
  - Crowd monitoring and management
  - Employee tracking
  - Transaction history and analytics
  - Responsive administrative interface

### 4. Add Employee (Pages/add-employee.html)
- **Description**: Form to add new employees to the system
- **Features**:
  - Employee information collection
  - Role assignment
  - Document upload capabilities
  - Form validation
  - Firebase database integration

### 5. Edit Employee (Pages/edit-employee.html)
- **Description**: Interface to modify existing employee information
- **Features**:
  - Pre-populated form with current employee data
  - Update functionality
  - Document management
  - Status changes (active/inactive)

### 6. Booking Details (Pages/booking-details.html)
- **Description**: Dashboard for managing fuel booking orders
- **Features**:
  - Booking list with filtering options
  - Status management (pending, confirmed, completed)
  - Customer information
  - Payment tracking

### 7. Transaction Details (Pages/transaction-details.html)
- **Description**: Comprehensive view of all financial transactions
- **Features**:
  - Transaction history with search and filter
  - Financial reporting
  - Export options
  - Transaction analytics

### 8. Safety Record (Pages/safety-record.html)
- **Description**: Interface for tracking safety-related incidents and compliance
- **Features**:
  - Incident logging
  - Compliance reporting
  - Safety metric visualization
  - Historical safety data

### 9. Subscription (Pages/Subscription.html)
- **Description**: Subscription management for premium services
- **Features**:
  - Subscription plan options
  - Payment integration
  - Subscription status tracking
  - Plan comparison

### 10. Alert System (Pages/alert.html)
- **Description**: Emergency and notification alert system
- **Features**:
  - Real-time alerts for critical situations
  - Notification preferences
  - Alert history
  - Staff notification system

## Firebase Integration Guide for Backend Developers

### 1. Project Setup
1. Create a new Firebase project at [Firebase Console](https://console.firebase.google.com/)
2. Register your app in the Firebase project
3. Copy the Firebase configuration to your project

```javascript
// Add this to your JavaScript file or in a script tag at the end of all HTML files
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "your-project-id.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project-id.appspot.com",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID",
  measurementId: "YOUR_MEASUREMENT_ID"
};

// Initialize Firebase
firebase.initializeApp(firebaseConfig);
```

### 2. Authentication Setup
FuelFlux uses Firebase Authentication for user management.

#### Implementation Steps:
1. Enable Email/Password authentication in Firebase Console
2. Implement sign-up functionality:

```javascript
// User registration
function registerUser(email, password, userData) {
  firebase.auth().createUserWithEmailAndPassword(email, password)
    .then((userCredential) => {
      // Save additional user data to Firestore
      return firebase.firestore().collection('users').doc(userCredential.user.uid).set({
        email: email,
        role: 'customer', // Default role
        name: userData.name,
        phone: userData.phone,
        created: firebase.firestore.FieldValue.serverTimestamp()
      });
    })
    .then(() => {
      // Success handling
    })
    .catch((error) => {
      // Error handling
    });
}
```

3. Implement login functionality:

```javascript
// User login
function loginUser(email, password) {
  firebase.auth().signInWithEmailAndPassword(email, password)
    .then((userCredential) => {
      // Redirect based on user role
      return firebase.firestore().collection('users').doc(userCredential.user.uid).get();
    })
    .then((doc) => {
      if (doc.exists && doc.data().role === 'admin') {
        window.location.href = 'Pages/Admin.html';
      } else {
        window.location.href = 'Pages/customer-dashboard.html';
      }
    })
    .catch((error) => {
      // Error handling
    });
}
```

4. Implement authentication state observer:

```javascript
// Auth state listener
firebase.auth().onAuthStateChanged((user) => {
  if (user) {
    // User is signed in
  } else {
    // User is signed out
  }
});
```

### 3. Firestore Database Structure
The application requires the following collections:

#### Collections:
1. **users**
   - Document ID: `{uid}`
   - Fields:
     - `email`: string
     - `name`: string
     - `phone`: string
     - `role`: string (admin, employee, customer)
     - `created`: timestamp

2. **employees**
   - Document ID: auto-generated
   - Fields:
     - `name`: string
     - `email`: string
     - `phone`: string
     - `position`: string
     - `salary`: number
     - `joiningDate`: timestamp
     - `documents`: array of strings (document URLs)
     - `status`: string (active/inactive)

3. **transactions**
   - Document ID: auto-generated
   - Fields:
     - `customerId`: string (reference to user)
     - `amount`: number
     - `date`: timestamp
     - `paymentMethod`: string
     - `fuelType`: string
     - `quantity`: number
     - `status`: string (completed, pending, failed)

4. **bookings**
   - Document ID: auto-generated
   - Fields:
     - `customerId`: string (reference to user)
     - `fuelType`: string
     - `quantity`: number
     - `scheduledTime`: timestamp
     - `status`: string (pending, confirmed, completed, cancelled)
     - `paymentStatus`: string
     - `createdAt`: timestamp

5. **safety_records**
   - Document ID: auto-generated
   - Fields:
     - `incidentType`: string
     - `description`: string
     - `date`: timestamp
     - `reportedBy`: string (reference to user)
     - `severity`: string (low, medium, high)
     - `status`: string (open, investigating, resolved)

6. **subscriptions**
   - Document ID: auto-generated
   - Fields:
     - `userId`: string (reference to user)
     - `plan`: string
     - `startDate`: timestamp
     - `endDate`: timestamp
     - `status`: string (active, expired, cancelled)
     - `paymentMethod`: string
     - `autoRenew`: boolean

### 4. Security Rules
Implement these Firestore security rules:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow users to read and write their own data
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Admin access
    match /{document=**} {
      allow read, write: if request.auth != null && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Employee records
    match /employees/{employeeId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Transactions
    match /transactions/{transactionId} {
      allow read: if request.auth != null && (
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin' ||
        resource.data.customerId == request.auth.uid
      );
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Bookings
    match /bookings/{bookingId} {
      allow read: if request.auth != null && (
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin' ||
        resource.data.customerId == request.auth.uid
      );
      allow create: if request.auth != null;
      allow update: if request.auth != null && (
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin' ||
        resource.data.customerId == request.auth.uid
      );
      allow delete: if request.auth != null && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

### 5. Firebase Storage Integration
For document uploads and image storage:

```javascript
// Upload employee document
function uploadEmployeeDocument(employeeId, file) {
  const storageRef = firebase.storage().ref();
  const documentRef = storageRef.child(`employees/${employeeId}/documents/${file.name}`);
  
  return documentRef.put(file).then(snapshot => {
    return snapshot.ref.getDownloadURL();
  }).then(downloadURL => {
    // Save download URL to employee record
    return firebase.firestore().collection('employees').doc(employeeId).update({
      documents: firebase.firestore.FieldValue.arrayUnion(downloadURL)
    });
  });
}
```

### 6. Real-time Updates
Implement listeners for real-time data changes:

```javascript
// Listen for real-time transaction updates
function listenForTransactions() {
  firebase.firestore().collection('transactions')
    .orderBy('date', 'desc')
    .onSnapshot((snapshot) => {
      snapshot.docChanges().forEach((change) => {
        if (change.type === 'added') {
          // Handle new transaction
        }
        if (change.type === 'modified') {
          // Handle modified transaction
        }
        if (change.type === 'removed') {
          // Handle removed transaction
        }
      });
    });
}
```

### 7. Cloud Functions (Advanced)
For implementing server-side logic like:
- Transaction validation
- Automatic email notifications
- Schedule processing
- Data aggregation for analytics

Example Cloud Function:

```javascript
// In your Firebase Cloud Functions index.js
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp();

exports.processBooking = functions.firestore
  .document('bookings/{bookingId}')
  .onCreate((snap, context) => {
    const newBooking = snap.data();
    
    // Send confirmation email
    const userRef = admin.firestore().collection('users').doc(newBooking.customerId);
    
    return userRef.get().then(userDoc => {
      if (!userDoc.exists) {
        throw new Error('User not found');
      }
      
      const user = userDoc.data();
      
      // Here you would integrate with an email service
      console.log(`Sending confirmation email to ${user.email} for booking ${context.params.bookingId}`);
      
      // Update booking status
      return snap.ref.update({
        status: 'confirmed',
        lastUpdated: admin.firestore.FieldValue.serverTimestamp()
      });
    });
  });
```

### 8. Deployment
1. Configure Firebase Hosting:
   ```
   firebase init hosting
   ```
2. Build and deploy:
   ```
   firebase deploy
   ```

## Development Guidelines
1. Always use Firebase SDK version 9.0.0 or higher
2. Maintain separation of concerns between UI and data logic
3. Implement proper error handling for all Firebase operations
4. Use Firebase emulators for local development
5. Set up continuous integration for testing before deployment
6. Follow the existing project structure and naming conventions
7. Implement proper user authorization checks in both client and server code

## Testing
1. Use Firebase Auth Emulator for authentication testing
2. Set up Firestore emulator for database testing
3. Test security rules using the Firebase Rules Simulator
4. Test Cloud Functions locally using the Functions Emulator

## Performance Considerations
1. Implement proper indexing for Firestore queries
2. Use batched writes for bulk operations
3. Optimize image uploads with compression before storage
4. Implement proper caching strategies for frequently accessed data

## Security Best Practices
1. Never expose Firebase API keys in client-side code
2. Always validate data on the server using Firebase Security Rules
3. Implement proper authentication checks for all operations
4. Use Firebase App Check to prevent abuse
5. Regularly audit and update security rules 