UserApp SDK for Android
=======================

This SDK adds user authentication and management to your app with [UserApp](https:/www.userapp.io). Then easily integrate your users with MailChimp, SendGrid, Mixpanel, etc. with just one click.

## Getting Started

**Download the SDK**

Download and include the [userapp-android.jar](https://github.com/userapp-io/userapp-android/raw/master/userapp-android.jar) file into your project.

**Include dependencies**

* [org.json](http://mvnrepository.com/artifact/org.json/json)

* [google-gson](https://code.google.com/p/google-gson/)

**Configure it**

Open your `AndroidManifest.xml` and add your UserApp App Id like this:

    <application ... >
        <meta-data android:name="userapp.AppId" android:value="YOUR-APP-ID" />
        ...
    </application>

[How do I find my App Id?](https://help.userapp.io/customer/portal/articles/1322336-how-do-i-find-my-app-id-)

**Add permissions**

Request Internet access in your manifest file, for example:

    <manifest ... >
        <uses-permission android:name="android.permission.INTERNET" />
        ...
    </manifest>

## Intro

This SDK integrates your Android app with UserApp by adding logic for authentication such as login and signup. It also keeps track of sessions, the logged in user, etc.

The SDK is focused on the use of fragments to show a login or signup form to the user when he/she is not logged in. And will automatically show it based on the session state. It also takes care of setting up persistent sessions so the user only would have to login once.

There are 3 main parts of this SDK that you should know about:

1. **UserApp.Session**
  
  This class takes care of everything related to the session; login, logout, events, etc. This should be created as an instance variable and have a life cycle of `onPause()` and `onResume()`.

2. **UserApp.UIHelper**

  This class helps you to show/hide the right fragment. It connects to your session and listens for state changes and show the login fragment when the user needs to log in. It can be created using the `session.createUIHelper()` which will automatically bind it to your session.
  
3. **AuthFragment**
  
To facilitate the creation of login and signup forms, this class can be extended to take care of the most of your bindings to UserApp. It has a few events that can be overriden to take more control over it.

## Log In

The login screen must be placed within a fragment inside your main activity.

Start with creating a new layout called `fragment_login`. The layout should include text fields for username and password, a login button, a progress loader, and a text for displaying errors.

Then create a new fragment called `LoginFragment` that extends
`AuthFragment` (*io.userapp.client.android.AuthFragment*) and inflates the `fragment_login` layout.

Now you need to bind the form to UserApp by calling the method `setupLoginForm()` on the superclass. Place this
code in the `onCreateView` function:

    super.setupLoginForm(view, R.id.login, R.id.password, R.id.login_button);

The first parameter is the view of the fragment, the second is the username field ID, then the password field ID, and last the ID of the login button. 

Override the methods `onLoginStart` and `onLoginCompleted` to show error messages and a progress loader (optional).

    @Override
    public Boolean onLoginStart(String login, String password, Boolean isSocialLogin) {
        // Called before login, e.g. show the loader here
        ...

        // Return true to complete the login
        return true;
    }
    
    @Override
    public void onLoginCompleted(Boolean authenticated, Exception exception) {
        // Called after login, e.g. hide the loader here and display errors
        ...
    }

The code should now look something like this:

    public class LoginFragment extends AuthFragment {
    	@Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
            // Inflate the layout for this fragment
            View view = inflater.inflate(R.layout.fragment_signup, container, false);

            // Setup the login form with bindings to UserApp
            super.setupLoginForm(view, R.id.login, R.id.password, R.id.login_button);

            return view;
        }

        @Override
        public Boolean onLoginStart(String login, String password, Boolean isSocialLogin) {
    	    // Show loader when waiting for server
    	    getView().findViewById(R.id.login_form).setVisibility(View.GONE);
    	    getView().findViewById(R.id.login_status).setVisibility(View.VISIBLE);
    	
    	    // Return true to complete the login
            return true;
        }
    
        @Override
        public void onLoginCompleted(Boolean authenticated, Exception exception) {
            // Hide the loader
            getView().findViewById(R.id.login_form).setVisibility(View.VISIBLE);
            getView().findViewById(R.id.login_status).setVisibility(View.GONE);
		
            if (exception != null) {
                // Show an error message
                ((TextView) getView().findViewById(R.id.error_text)).setText(exception.getMessage());
            } else {
                // Clear the message
                ((TextView) getView().findViewById(R.id.error_text)).setText("");
            }
        }
    }
    
The login fragment is ready, add it to your main activity's layout:

    <RelativeLayout ... >
        <fragment
            android:id="@+id/loginFragment"
            android:name="com.example.demo.LoginFragment"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:layout_alignParentLeft="true"
            android:layout_alignParentTop="true" />

        ...
    </RelativeLayout>

Also create a main fragment that will be shown after a successful login.

And last we will add the code to show/hide the right fragments; the login fragment when the user needs to log in, 
and the main fragment when the user has logged in. In your `MainActivity` class, begin with creating instances of `UserApp.Session and `UserApp.UIHelper`.

Create a new session instance:

    UserApp.Session session;
    
Create a new UIHelper instance:

    UserApp.UIHelper uiHelper;

These should be placed as instance variables in the class, and you should call `session.onResume()` and `session.onPause()` in the activity's or fragment's life cycle events. In this case you will be using the UIHelper and then you will call these methods on that object instead (see below).

Then call the method `session.createUIHelper()` in `onCreate`, which takes the login fragment ID as the first argument, the main fragment ID as the second, and the rest should be one argument for each additional fragments you have in your activity. In this case we will only have two arguments since we don't have any more fragments. The return will be a new instance of a `UIHelper`.

Also make sure to call `uiHelper.onResume()` and `uiHelper.onPause()` in the events as the example below.

    public class MainActivity extends FragmentActivity {
        // Instances for session and its UI helper
        UserApp.Session session;
        UserApp.UIHelper uiHelper;
	
        @Override
        protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_main);
		
	    // Initiate the session and create a new UI helper bound to it
	    session = new UserApp.Session(this);
	    uiHelper = session.createUIHelper(R.id.loginFragment, R.id.mainFragment);
        }
	
        @Override
        public void onResume() {
	    super.onResume();
	    uiHelper.onResume();
        }

        @Override
        public void onPause() {
	    super.onPause();
	    uiHelper.onPause();
        }
    }

If you run your app you will see the login screen. If you log in you will either see an error message or you will see the main fragment. The session will be permanently stored and is remembered until you manually log out.

## Log Out

When you've got login to work, you probably want to have the option to log out. All you have to do is to call the method `session.logout()`. Here's an example of a logout menu in the `MainActivity` class:

    public class MainActivity extends FragmentActivity {
        UserApp.Session session;
        ...

        @Override
        public boolean onCreateOptionsMenu(Menu menu) {
            getMenuInflater().inflate(R.menu.main, menu);
            return true;
        }
	
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
            // Handle presses on the action bar items
            switch (item.getItemId()) {
                // Logout action
                case R.id.action_logout:
                    session.logout();
	            return true;
                default:
                    return super.onOptionsItemSelected(item);
            }
        }
        
        ...
    }

This will automatically end the session and show the login fragment. If you have multiple activities you will need to show the main activity after you called `session.logout()`.

## Social Login

If you rather want your users logging in with their social account (or both), it's easy to add login with Facebook, etc. using the SDK.

Start by adding a new button to your login fragment layout:

    <LinearLayout ... >
        ...

        <Button
            android:id="@+id/facebook_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:paddingLeft="32dp"
            android:paddingRight="32dp"
            android:text="Log In With Facebook" />

        ...
    </LinearLayout>

Then go to your `LoginFragment` class and call `super.setupSocialLogin()` from `onCreateView`. The first argument is the view, the second is the button ID you want to attach, and the third is the Provider Id of the provider your want to attach, e.g. `"facebook"`.

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_login, container, false);
        
        ...
        
        // Attach the Facebook button
        super.setupSocialLogin(view, R.id.facebook_button, "facebook");

        ...
    }

## Sign Up

Great, login is done! But users may still want to sign up the regular way, without using their social network accounts. This is easily fixed with a new fragment, just as with the login fragment.

Start with creating a new layout called `fragment_signup`. Add 3 text boxes; one for username, one for name, one for email, and one for password. Use `android:tag` to specify which field it should be mapped to, e.g. `android:tag="password"`. Also create a button and a progress loader.

Create a new fragment called `SignupFragment` that extends
`AuthFragment` (*io.userapp.client.android.AuthFragment*) and inflates the `fragment_signup` layout.

Now you need to bind the form to UserApp by calling the method `setupSignupForm()` on the superclass:

    super.setupSignupForm(view, R.id.signup_button, R.id.email, R.id.login, R.id.password);

The first parameter is the view of the fragment, the second is the signup button field ID, then the IDs of all the input fields. 

Override the methods `onSignupStart` and `onSignupCompleted` to show error messages and a progress loader. `onSignupStart` can also be used
to fill the user profile with additional information before submitting.

    @Override
    public User onSignupStart(User user) {
    	// Show loader when waiting for server
    	...
    	
    	// Alter the user profile here
    	...
    
    	// Return the user to complete the signup
    	return user;
    }
    
    @Override
    public void onSignupCompleted(User user, Boolean verified, Exception exception) {
		if (exception != null) {
			// Hide the loader
			...
			
			// Show an error message
			...
		} else {
			// Clear the error message
			...
			
			// Need to verify the email address?
			if (verified == false) {
				// Hide the loader
				...
				
				// Show message that an email has been sent to be verified
				...
			} else {
				// Call the superclass to log in the signed up user
				super.onSignupCompleted(user, verified, exception);
			}
		}
	}

The code should now look something like this:

    public class SignupFragment extends AuthFragment {
	
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
            // Inflate the layout for this fragment
            View view = inflater.inflate(R.layout.fragment_signup, container, false);
        
            // Setup the login form with bindings to UserApp
            super.setupSignupForm(view, R.id.signup_button, R.id.email, R.id.login, R.id.password);
        
            return view;
        }

        @Override
		public User onSignupStart(User user) {
	    	// Show loader when waiting for server
	    	getView().findViewById(R.id.signup_form).setVisibility(View.GONE);
	    	getView().findViewById(R.id.signup_status).setVisibility(View.VISIBLE);
	    	
	    	// Set subscription
	    	user.subscription = new Subscription();
	    	user.subscription.price_list_id = "IEiVxVpxSxy1xE8oIzsSeA";
	    	user.subscription.plan_id = "CBPJdFORQ-qsefa7LxrX-A";
	    	
	    	// Set age property
	    	user.properties.put("age", new Property(42, true));
	    
	    	// Return the user to complete the signup
	    	return user;
	    }
	    
		@Override
		public void onSignupCompleted(User user, Boolean verified, Exception exception) {
			if (exception != null) {
				// Hide the loader
				getView().findViewById(R.id.signup_form).setVisibility(View.VISIBLE);
				getView().findViewById(R.id.signup_status).setVisibility(View.GONE);
				
				// Show an error message
				((TextView) getView().findViewById(R.id.error_text)).setText(exception.getMessage());
			} else {
				// Clear the error message
				((TextView) getView().findViewById(R.id.error_text)).setText("");
				
				// Need to verify the email address?
				if (verified == false) {
					// Hide the loader
					getView().findViewById(R.id.signup_form).setVisibility(View.VISIBLE);
					getView().findViewById(R.id.signup_status).setVisibility(View.GONE);
					
					((TextView) getView().findViewById(R.id.error_text)).setText("An email has been sent to your inbox.");
				} else {
					// Call the superclass to log in the signed up user
					super.onSignupCompleted(user, verified, exception);
				}
			}
		}

    }
    
The signup fragment is ready, add it to your main activity's layout:

    <RelativeLayout ... >
        <fragment
            android:id="@+id/signupFragment"
            android:name="com.example.demo.SignupFragment"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:layout_alignParentLeft="true"
            android:layout_alignParentTop="true" />

        ...
    </RelativeLayout>

Now go to your main activity and add it to the `createUIHelper` arguments (this will hide it when the login or main fragment is shown):

    @Override
    protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		// Initiate the session and create a new UI helper bound to it
		session = new UserApp.Session(this);
		uiHelper = session.createUIHelper(R.id.loginFragment, R.id.mainFragment, R.id.signupFragment);
	}

You would need to add a link from the login fragment so your users would be able to access your signup screen. 
See the demo app for an example on how to do that.

## Back-end

To connect your Android app to a back-end API, use `session.current.token` to get the session token and send that along with all HTTP requests. And then on the back-end, get that token and use UserApp's [token.heartbeat()](https://app.userapp.io/#/docs/token/#heartbeat) or [user.get()](https://app.userapp.io/#/docs/user/#get) to verify that the user is authenticated. The result should then be cached to reduce round-trips to UserApp.

## State Changes

Sometimes it would be good to know when a user logs in or logs out. One use-case would be in the main fragment to update the UI with the user's name.

    session = new UserApp.Session(this.getActivity(), new UserApp.Session.StatusCallback() {
	    @Override
	    public void call(Boolean authenticated, Exception exception) {
	        if (authenticated) {
	            ((TextView) view.findViewById(R.id.welcome_text)).setText("Welcome " + session.user.first_name);
	        }
	    }
	});

You can also use `session.addCallback()` to add more callbacks.

This will also be called when the app starts and the user has already logged in before.

**Note:** Always call `session.onPause()` and `session.onResume()` in the fragment's or activity's life cycle events.

## Reload User Profile

The user profile is cached locally at login, but you can easily reload it by calling:

    session.reloadUser(new UserApp.Session.UserCallback() {
	    @Override
	    public void call(User user, Exception exception) {
	        if (exception == null) {
	        	((TextView) view.findViewById(R.id.welcome_text)).setText("Welcome " + user.first_name);
	        } else {
	        	System.out.println("ERROR LOADING USER: " + exception.getMessage());
	        }
	    }
	});

You can also listen for profile update events with a callback:

    session.addUserCallback(new UserApp.Session.UserCallback() {
	    @Override
	    public void call(User user, Exception exception) {
	        if (exception == null) {
	        	...
	        } else {
	        	...
	        }
	    }
	});

## Save User Profile

If you want to change and save the user's profile, make the changes in the user object and then call `session.save()` to save it, like this:

    // Save user changes
    session.user.first_name = "John";
	session.user.properties.get("age").value = 34;
	session.user.properties.get("age").override = true;
	session.saveUser(session.user, new UserApp.Session.UserCallback() {
	    @Override
	    public void call(User user, Exception exception) {
	        if (exception == null) {
	        	System.out.println("USER SAVED");
	        } else {
	        	System.out.println("ERROR SAVING USER: " + exception.getMessage());
	        }
	    }
	});

If you want to save a new user (i.e. "sign up"), just create a new `User` object and leave `user_id` empty and then save it.

When you save a user, all the user callbacks will be called (see *Reload User Profile*).

## Check Permissions

To check if a logged in user has a specific set of permissions, use this code:

    if (session.hasPermission("admin")) {
		// Is admin
	} else {
		// Is not admin
	}

You can also check for features using `session.hasFeature()`. Use a space delimited string to insert multiple permissions or features.

**Note:** These checks should be wrapped in a state callback to make sure that the user is logged in and loaded.

## Personalization

When a user has logged in you probably want to personalize their experience. Use the session object to get the logged in user's profile.

    User user = session.user;

## Multiple Activities

If you have multiple activities in your project, you would need to listen for session state changes in all of them and show the main activity if the user has logged out. It could also be a good idea to set the main activity to only have a single instance.

## Example

See [example/](https://github.com/userapp-io/userapp-ember/tree/master/example) for a demo app based on the Ember Starter Kit and Bootstrap.

## Help

Contact us via email at support@userapp.io or visit our [support center](https://help.userapp.io). You can also see the [UserApp documentation](https://app.userapp.io/#/docs/) for more information.

## License

MIT, see LICENSE.