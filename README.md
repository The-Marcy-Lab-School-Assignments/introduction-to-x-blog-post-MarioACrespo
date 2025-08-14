# Secure Login with Google Auth: A Developer's Introduction
By Mario Crespo

## Why Google Auth Belongs in Every Developer's Toolkit
When I was planning my Capstone project I knew I wanted to use Google Calendar. To access each user's personal calendar I needed a way for users to sign in with their Google accounts. That is where Google Auth came in and it also had the added bonus of me not having to make a whole login system from scratch

Google Auth lets your app confirm a user's identity without ever handling their password adding another time save of not having to secure the password. Instead of storing credentials yourself your app receives a secure token from Google. Think of it like showing a passport at the airport. You never created it but it is trusted because it comes from a verified source. Whenever you go onto a site or an app notice how often you see the option to use Google to log in

## Setting Up Google Auth

### Steps:

**Create a project in Google Cloud Console**
Start by creating a project in Google Cloud Console. This gives you the information you need to use like CLIENT_ID and 

**Enable Google Identity Services**
Turn on Google Identity Services for your project. This lets your app use Google accounts to sign in the users

**Register your app's authorized redirect URIs**
Add the redirect URIs for your app. These are the test and the deployed sites that google will need to allow you and other users to use their resources

I had a lot of trouble and needed a lot of help getting started since I wasn't used to this new and far bigger set of resources, so don't forget to ask for help and look things up

This .env variable is your main access to the Google Authentication
```
GOOGLE_CLIENT_ID=your_client_id_here
```

To add the Google Auth into a project, you need both the frontend and the backend packages. On the frontend, you need @react-oauth/google. On the backend, you need google-auth-library. Installing them is simple:

```bash
# Frontend
npm install @react-oauth/google

# Backend
npm install google-auth-library
```

Once installed, you can begin making the frontend Google Login button and backend token verification. The frontend renders a Sign In button and sends the ID token to your backend once a user logs in. Here's a simplified example of it:

```javascript
import React from 'react';
import { GoogleLogin } from '@react-oauth/google';

function App() {
  // This function runs when the user successfully logs in with Google
  const handleLoginSuccess = (response) => {
    // Send the ID token to the backend for verification
    fetch('http://localhost:5000/api/auth/google', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      // The token contains the user's Google identity info
      body: JSON.stringify({ token: response.credential }),
    })
      .then((res) => res.json())
      .then((data) => {
        // Backend sends back verified user info
        console.log('User authenticated:', data);
      })
      .catch((error) => {
        console.error('Error during authentication:', error);
      });
  };

  return (
    <div>
      {/* Renders Google Sign-In button */}
      <GoogleLogin onSuccess={handleLoginSuccess} onError={() => console.log('Login Failed')} />
    </div>
  );
}

export default App;
```

In this example, the GoogleLogin component renders a button. When the user signs in, the handleLoginSuccess function sends the ID token to the backend, which will verify it. While on the backend side, the ID token is verified to ensure if it's authentic before any user actually logs in. Here's a quick example of what it would look like in code:

```javascript
const express = require('express');
const { OAuth2Client } = require('google-auth-library');

const app = express();
const client = new OAuth2Client('YOUR_GOOGLE_CLIENT_ID'); // Unique ID for your app

app.use(express.json());

app.post('/api/auth/google', async (req, res) => {
  const { token } = req.body; // Receive the ID token sent from the frontend

  try {
    // Verify the token with Google's library
    const ticket = await client.verifyIdToken({
      idToken: token,
      audience: 'YOUR_GOOGLE_CLIENT_ID', // Ensures token is meant for this app
    });

    const payload = ticket.getPayload(); 
    // Payload contains verified user info (email, name, picture, etc.)
    res.json({ message: 'User authenticated', user: payload });
  } catch (error) {
    // If verification fails, return an error
    console.error('Error verifying token:', error);
    res.status(401).json({ error: 'Invalid token' });
  }
});

app.listen(5000, () => {
  console.log('Server running on http://localhost:5000');
});
```

The backend receives the token, verifies it using Google's library, and extracts the user's information. Only the verified users can proceed, which ensures secure authentication.

## Final Thoughts
In my Capstone, using Google Auth saved me from building a password and login system from scratch while also completing the reason I needed it for, accessing each user's google calendar.

### Plan for Real World Scenarios
When I first used Google Auth I ran into a big problem because I did not check when Google would update their servers and security protocols. My app would not deploy because Google stopped accepting the older code it was built on. Always research what you are working with and stay aware of any plans of change so you will not end up with an unforeseen problem.

### Test in Multiple Environments Early
Do not wait until the end of development to test Google Auth under real conditions. I made that mistake, and problems like redirect URI mismatches and HTTPS requirements hit me right before launch. If I had tested earlier, I could have caught them when fixes were quick and I had more time, instead of scrambling at the last minute and worrying if a fix would break more than fix.

### Monitor API Usage and Error Logs
Google provides dashboards to track API calls, errors, and unusual activity. Take some time to check these regularly. You might spot patterns like failed token exchanges that will help you find any hidden errors or soon to be problems you may have missed.

## Learning Resources:
- [Google Identity Services Web Guide](https://developers.google.com/identity/gsi/web)
- [Google Auth Library for Node](https://github.com/googleapis/google-auth-library-nodejs)
- [React OAuth Google Package](https://www.npmjs.com/package/react-oauth-google)
- [OAuth 2.0 Overview](https://oauth.net/2/)
- [Google OAuth Playground](https://developers.google.com/oauthplayground)
