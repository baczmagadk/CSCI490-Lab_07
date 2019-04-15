# CSCI490-Lab_07

This lab demostrates Firebase services from Google. In order to bring in these powerful features in the finished app, you will need to create a project using the Firebase Console. Instructions to do this are next. If you don’t have a Google account yet, you can create one [here](https://accounts.google.com/signup/v2/webcreateaccount?flowName=GlifWebSignIn&flowEntry=SignUp). 


## Create Project in Firebase Console ##
* Start by navigating to the [Firebase Console](https://console.firebase.google.com/) webpage.
* Select "Add PROJECT" and name your project. Fill in the name and agree to the terms. Click 'Create Project'.
* When the project is ready, click 'Continue'.

## Get started by adding Firebase to your app ##
* Click the Andoid icon to add an Android app to the Firebase project
* Add your package name for the app. This can be found in the Manifest file.
* Click 'Register App'.
* You’ll need to add the debug signing certificate too because you’ll implement Google Sign-In for authentication in FriendlyChat. The SHA-1 is a type of hash representation for the debug keystore, which you can get with the keytool command line tool. Which is a long way of saying, the debug keystore is a bunch of letters and numbers, which you should keep secret, that identifies your computer.

On Windows, open the Command Prompt program. You can do this by going to the Start menu
````
keytool -exportcert -list -v \
-alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
````
On Mac/Linux, open the Terminal and paste
````
keytool -exportcert -list -v \
-alias androiddebugkey -keystore ~/.android/debug.keystore
````
* Copy and paste the 'SHA1' string and paste into the Firebase Console certificate section. 
* Download the google-services.json file and add to the app/ directory of your project.
* Add the Firebase SDK to your project by following the directions from the Firebase Console.
* Make sure you 'Sync Now'

## Adding Firebase Realtime Database to your app ##
* Firebase Realtime Database must be added to your app. Add ***implementation 'com.google.firebase:firebase-database:16.0.1'*** to your app/build.gradle file in the ***dependencies*** section.
* More information can be found [here](https://firebase.google.com/docs/android/setup), however I have not had much luck in using this page. Version numbers seem to be off and never compatible with what I am doing. 

## Write messages to Firebase Database ##
* Add the Database references to you ***MainActivity.java***
````
private FirebaseDatabase mFirebaseDatabase;
private DatabaseReference mMessagesDatabaseReference;
````
The FirebaseDatabase object will be a reference to your entire FirebaseDatabase and the DatabaseReference object will be a reference to the ***messages*** collection. 

* Add the following code in your ***onCreate()*** method:
````
mFirebaseDatabase = FirebaseDatabase.getInstance();
mMessagesDatabaseReference = mFirebaseDatabase.getReference().child("messages");
````
* Ensure there is an OnClickListener attached to the button
````
// Send button sends a message and clears the EditText
mSendButton.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View view) {
       // TODO: Send messages on click


       // Clear input box
       mMessageEditText.setText("");
   }
});
````
Within the onClick method, let’s create a FriendlyMessage object for the message that the user typed in. The FriendlyMessage object has three instance variables: A String for the user’s name, A String for the text of the message A String for the URL of the photo if it’s a photo message.

````
FriendlyMessage friendlyMessage = new FriendlyMessage(mMessageEditText.getText().toString(), mUsername, null);
mMessagesDatabaseReference.push().setValue(friendlyMessage);
````


In this case, we’re only sending text messages for now (we will implement photo-messaging later), so we’ll create a FriendlyMessage object with all the fields except for photoUrl, which will be null.

* At this point, return to the Firebase Console and click on the 'Database' tab and click 'Create database'. Chose 'Start in Test Mode' and click 'Enable'.
* You'll notice that the default view if for a Cloud Firestore. In the Database dropdown, choose 'Realtime Database'.
* Run the app and try to send a message while monitoring Logcat.
* What did you notice? You should have seen that you were denied permission to write to the database.
* In the Firebase Console under Database (make sure you are looking at Realtime Database) and click the 'Rules' tab. Change the permissions to look like:
````
{
  /* Visit https://firebase.google.com/docs/database/security to learn more about security rules. */
  "rules": {
    ".read": true,
    ".write": true
  }
}
````
NOTE: This is just for development. The database is now open and not secure. We will fix this later on...

## Read messages from Firebase Database ##

* In order to receive data/updates from Firebase, we must set up a Listener. Firebase has created an API through ChildEventListener. You mush implement this interface by @Override five methods.
* Add a ChildEventListener as a member variable:
````
private ChildEventListener mChildEventListner;
````


* Set up the ChildEventListener
````
private void attachDatabaseReadListener() {
        if(mChildEventListner == null) {
            mChildEventListner = new ChildEventListener() {
                @Override
                public void onChildAdded(DataSnapshot dataSnapshot, String s) {
                    // Method that gets called whenever a message gets inserted
                    // It is also triggered for every message that is in the database
                    // when the listener is first attached

                }

                @Override
                public void onChildChanged(DataSnapshot dataSnapshot, String s) {
                    // Method that gets called when an existing message is changed

                }

                @Override
                public void onChildRemoved(DataSnapshot dataSnapshot) {
                    // Method that gets called when an existing message is deleted

                }

                @Override
                public void onChildMoved(DataSnapshot dataSnapshot, String s) {
                    // Method that gets called when an existing message changes position
                }

                @Override
                public void onCancelled(DatabaseError databaseError) {
                    // Method that gets called when some error occurred
                }
            };

            mMessagesDatabaseReference.addChildEventListener(mChildEventListner);
        }
    }
````
* In the onCreate(), call attachDatabaseReadListener().

* You should now be able to send messages and see them appear in Firebase Console as well as displayed in your app.
### ~~~~~~~~~~~~~~~` Make sure this works before going further ~~~~~~~~~~~~~~~~ ###

## Adding Authentication ##
* In the Firebase Console, go back to the 'Rules' tab of the Realtime Database.
* Change the rule back to the original setting
````
{
  /* Visit https://firebase.google.com/docs/database/security to learn more about security rules. */
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
````
* Firebase simplifies the authentication steps by already providing different implementation depending on which types you choose to allow. These could be email/password, Facebook, Twitter, Google, Github, amoung others. FirebaseUI is a client that Google has created to seemlessly incorporate any that you choose.
* ***In Firebase Console*** click on the 'Authentication' tab.
* Click 'Set up sign-in method'.
* Click on and enable 'Email/password' and 'Google'.
* Navigate a browser tab to [Firebase Auth Doc](https://firebase.google.com/docs/auth/).
### Basic steps to add authenticating to our app ###
1. Add dependencies.
2. Add AuthStateListener.
3. Send unauthenticated users to authentication flow.
4. Sign in setup and sign out teardown.

* ***In Android Studio*** Change to the dependencies in your app/build.gradle file:
````
implementation fileTree(dir: 'libs', include: ['*.jar'])
implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    
implementation 'com.android.support:design:27.0.2'
    
implementation 'com.android.support:appcompat-v7:27.0.2'
implementation 'com.android.support:multidex:1.0.1'

implementation 'com.google.firebase:firebase-core:11.8.0'
implementation 'com.google.firebase:firebase-database:11.8.0'
implementation 'com.google.firebase:firebase-auth:11.8.0'
implementation 'com.google.firebase:firebase-storage:11.8.0'

implementation 'com.firebaseui:firebase-ui-auth:3.2.2'

// Displaying images
implementation 'com.github.bumptech.glide:glide:3.6.1'

testImplementation 'junit:junit:4.12'
androidTestImplementation 'com.android.support.test:runner:1.0.2'
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'

````
Once the libraries are added, we are ready to start coding. The app will now have two states, signed in and signed out. We need a way to determine which state our app is in. An ***AuthStateListener*** reacts to auth state changes.

* Add a FirebaseAuth and FirebaseAuth.AuthStateListener as member variables to MainActivity.java.
````
private FirebaseAuth mFirebaseAuth;
private FirebaseAuth.AuthStateListener mAuthStateListener;
````
In onCreate() directly under where you instantiate the Firebase Database, instantiate the FirebaseAuth object.
````
mFirebaseAuth = FirebaseAuth.getInstance();
````
At the end of onCreate() instantiate the AuthStateListner. As with other interfaces you have implemented on the fly, let the IDE do the work by auto completing.
* mAuthStateListener = new Auth.... ***press Enter***.
* @Override onPause() and onResume() methods.
* on mFirebaseAuth object add the mAuthStateListener in the onResume() and remove the listener in the onPause().
* We need to add some functionality to the AuthStateListener. In the ***onAuthStateChanged()*** 
````
FirebaseUser user = firebaseAuth.getCurrentUser();
 if(user != null) {
     // user is signed in

 } else {
     // user is signed out
     startActivityForResult(
             AuthUI.getInstance()
                     .createSignInIntentBuilder()
                     .setAvailableProviders(Arrays.asList(
                             new AuthUI.IdpConfig.GoogleBuilder().build()))
                     .build(),
             RC_SIGN_IN);
 }
 ````
 * Add the RC_SIGN_IN constant for startActivityForResult() method.
 ````
 public static final int RC_SIGN_IN = 1;
 ````
 * Add a Toast message in the ***if*** block 
 ````
 Toast.makeText(getApplicationContext(), "Signed in!", Toast.LENGTH_SHORT).show();
 ````
 
* Test the app to ensure you can at least log in
* Create two private void methods ***onSignedInInitialize(String username)*** and ***onSignedOutCleanup()***
````
private void onSignedInInitialize(String username) {
        
}
    
private void onSignedOutCleanup() {
        
}

````
* Inside the ***onSignedInIntialize()*** method, set mUsername to the username that is passed in.
* Move the line that calls ***attachDatabaseReadListener()*** to inside ***onSignedInIntialize()***
The app now attaches the database listener after the user has authenticated. Test the app and make sure that a test message now displays who sent it.
* Create a method to detach the DatabaseListener.

````
private void detachDatabaseReadListener() {
    if(mChildEventListner != null) {
        mMessagesDatabaseReference.removeEventListener(mChildEventListner);
        mChildEventListner = null;

    }
}
````
* Inside the ***onSignedOutCleanup()*** set mUsername to ANONYMOUS, clear the message adapter, and call the ***detachDatabaseListener()***
* @Override onResume() and call ***attachDatabaseReadListener()***.
* @Override onPause() and call ***detachDatabaseListener()***.
````
@Override
protected void onPause() {
   super.onPause();
   if(mAuthStateListener == null) {
      mFirebaseAuth.removeAuthStateListener(mAuthStateListener);
   }
   detachDatabaseReadListener();
   mMessageAdapter.clear();
}

@Override
protected void onResume() {
   super.onResume();
   mFirebaseAuth.addAuthStateListener(mAuthStateListener);
}
````
You might notice that there is a menu item for Signing out. You can write the code for this by @Overriding the ***onOptionsItemSelected(MenuItem item)*** method.
````
@Override
public boolean onOptionsItemSelected(MenuItem item) {
   switch (item.getItemId()) {
      case R.id.sign_out_menu:
          // sign out
          AuthUI.getInstance().signOut(this);
          return true;
      default:
          return super.onOptionsItemSelected(item);
   }

}
````
You may notice there is a slight bug. Once the user is logged out, there is no way to back out of the app. A slightly less buggy fix for this:
````
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
   super.onActivityResult(requestCode, resultCode, data);

   if(requestCode == RC_SIGN_IN) {
      if(resultCode == RESULT_OK) {
          Toast.makeText(getApplicationContext(), "Signed in!", Toast.LENGTH_SHORT).show();
      } else if(resultCode == RESULT_CANCELED) {
          Toast.makeText(getApplicationContext(), "Cancelled!", Toast.LENGTH_SHORT).show();
          finish();
      }
   }
}
````
## File Storage ##
* In Firebase Console, click on the 'Storage' tab.
* Click 'Get Started'.
* Add a folder called ***chat_photos***
* In Android Studio, set up the image picker
````
// ImagePickerButton shows an image picker to upload a image for a message
mPhotoPickerButton.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View view) {
       Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
       intent.setType("image/jpeg");
       intent.putExtra(Intent.EXTRA_LOCAL_ONLY, true);
       startActivityForResult(Intent.createChooser(intent, "Complete action using"), RC_PHOTO_PICKER);
   }
});
````
* Under the 'RC_SIGN_IN' int, create 'RC_PHOTO_PICKER'
````
private static final int RC_PHOTO_PICKER =  2; 
````
* Create a member variable called ***FirebaseStorage mFirebaseStorage*** and instantiate it like you did FirebaseAuth and FirebaseDatabase.
* Create a member variable called ***StorageReference mChatPhotosStorageReference*** and instantiate it like you did the DatabaseReference but pass in the folder you created in Firebase Console.
* Add an ***else if*** to the onActivityResult() method.
````
else if(requestCode == RC_PHOTO_PICKER && resultCode == RESULT_OK ) {

}
````
This is where we will handle what to do after the user chooses a picture.
The URI will come in as ***data.getData()***
* Inside the else if block you just created, you will need three lines of code. One to get the URI, one to get a reference to the the file location we will be putting the file into, and lastly to put the file in Firebase.
````
Uri selectedURI = data.getData();
            
// Get a reference to store file in chat_photos/<filename>
StorageReference photoReference = mChatPhotosStorageReference.child(selectedURI.getLastPathSegment());

// Put file in Firebase
photoReference.putFile(selectedURI);
````
Finally, we need to add a SuccessListner to the file push. This will grab the download URL and send it to the messages database and allow the image to be shown.
* Modify the ***putFile*** as follows:
````
// Put file in Firebase
photoReference.putFile(selectedURI).addOnSuccessListener(new OnSuccessListener<UploadTask.TaskSnapshot>() {
    @Override
    public void onSuccess(UploadTask.TaskSnapshot taskSnapshot) {
        Uri downloadUrl = taskSnapshot.getDownloadUrl();

        FriendlyMessage message = new FriendlyMessage(null, mUsername, downloadUrl.toString());
        mMessagesDatabaseReference.push().setValue(message);
    }
});
````
You will need to download a photo from web on virtual devices, but you can now test the functionality

## Firebase Messaging ##
* Add Firebase Messaging dependency and Sync
````
implementation 'com.google.firebase:firebase-messaging:11.8.0'
````
* Ensure app is not in Foreground.
* In the Firebase Console, click on the ***Cloud Messaging*** tab.
* Click on 'Send your first message'.
* Fill in a Notification and click 'Next'
* Target 'edu.cofc.briggs.myapplication'.

