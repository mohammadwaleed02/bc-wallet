# Fix for "Create Profile" Button Not Working

## Problem
The green "Create Profile" button in the Teacher Registration modal (line 840) calls `registerTeacher()` but this function is **not implemented** in the Vue setup.

## Solution
Add the `registerTeacher` function inside the Vue `setup()` method (starting at line 997).

### Location to Add the Function
**After line 1000-ish** - somewhere inside the `setup()` method, add:

```javascript
const registerTeacher = async () => {
    try {
        pinError.value = '';
        
        // Validation
        if (!teacherRegisterForm.value.displayName || !teacherRegisterForm.value.username || !teacherRegisterForm.value.password) {
            pinError.value = 'Please fill in all fields';
            return;
        }
        
        // Check if username already exists
        const teachersRef = collection(db, 'teachers');
        const q = query(teachersRef, where('username', '==', teacherRegisterForm.value.username));
        const snapshot = await getDocs(q);
        
        if (snapshot.size > 0) {
            pinError.value = 'Username already taken. Please choose another.';
            return;
        }
        
        // Create new teacher document in Firestore
        const newTeacher = {
            displayName: teacherRegisterForm.value.displayName,
            username: teacherRegisterForm.value.username,
            password: teacherRegisterForm.value.password, // ⚠️ SECURITY: Consider using proper authentication!
            createdAt: new Date(),
            students: []
        };
        
        const docRef = await addDoc(teachersRef, newTeacher);
        
        // Log them in
        currentTeacher.value = { id: docRef.id, ...newTeacher };
        localStorage.setItem('currentTeacherId', docRef.id);
        
        // Reset form and close modal
        teacherRegisterForm.value = { displayName: '', username: '', password: '' };
        showPinModal.value = false;
        
        showToast('Teacher profile created successfully!', 'fa-circle-check');
    } catch (err) {
        console.error('Registration error:', err);
        pinError.value = 'Error creating teacher profile: ' + err.message;
    }
};
```

### Also Update the Import Statement
**Change line 957 from:**
```javascript
import { getFirestore, collection, doc, addDoc, updateDoc, deleteDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
```

**To:**
```javascript
import { getFirestore, collection, doc, addDoc, updateDoc, deleteDoc, onSnapshot, query, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
```

### Don't Forget to Return the Function
Make sure `registerTeacher` is included in the object returned by `setup()`. Search for the `return {` statement and add:
```javascript
registerTeacher,
```

## ⚠️ Security Warning
Storing passwords in **plaintext** in Firestore is a **major security risk**. Consider:
1. Using Firebase Authentication instead
2. Hashing passwords with bcryptjs
3. Never storing passwords client-side

