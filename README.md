# WakandaEmailCustomLogin
An example of Wakanda (2.0+) for a backend with a custom login based on email

The simple model 

![alt text](https://github.com/rmello4d/WakandaEmailCustomLogin/blob/master/model.png)

Note : 
1. This login uses the HA1key to save your password 
2. The password is actually a calculated field, it does not exist in the database, the value is stored in the HA1Key attribute
2. Check the file testLogin.js for examples


Understanding the example: 

1. Bootstrap file
You have to set the loginListener method on the bootstrap file 

``` directory.setLoginManager( "/login/login", "Admin" );```


2. Add the module login to export the login method

```javascript
//Wakanda Login Listener
var login = function (emailAddress, password) {

	var connectTime;
	var myUser = ds.User({email:emailAddress}); // Get the user

	if (myUser === null) {
		return false;
	} else {
		//we will handle login
		if (myUser.validatePassword(password)) {
			
			connectTime = new Date();
			return {
				ID: myUser.ID,
				email: myUser.email,
				storage: {time: connectTime}
			}
			
		} else {
			return {error: 1024, errorMessage: "invalid login"};
		}
		
	}
};

module.exports.login = login;
```

3. Add User events for the password (get and set) 

```javascript
model.User.password.onGet = function() {
	return "*****";
};
model.User.password.onSet = function(value) {
	this.HA1Key = directory.computeHA1(this.ID, value);
};
```

4. Add user method to validate the password
```javascript
model.User.entityMethods.validatePassword = function(password) {
	var ha1 = directory.computeHA1(this.ID, password);
	return (ha1 === this.HA1Key); //true if validated, false otherwise.

};
```

5. Add user event to validate the email address (it will throw an error if the email is not valid)
```javascript
model.User.email.events.validate = function(event) {
	var err, emailRegexStr, isValid;
	//Check the email to see if it's valid.
	if (this.email !== null) {
		emailRegexStr = /^[a-zA-Z0-9.-_]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$/;
		isValid = emailRegexStr.test(this.email);
		if (!isValid) {
			return {error: 1, errorMessage: "Email is invalid."};
		}
	}
    else {
        return {error: 0}; //Same as no error
    } 

};
```


Now just use the login on the client side as usual 
```javascript
this.wakanda.directory.login(userEmail, userPassword)
	    .then((result) => {	... }) 
```      

