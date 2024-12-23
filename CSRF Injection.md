# Website Vulnerabilities - CSRF
> July 3rd, 2023
----------------------------------------------------------------

## Cross-Site Request Forgery

CSRF is an attack that forces an end user to execute unwanted actions on a web applicaiton in which they're currently authenticated. CSRF attacks target specifically target state-changing requests, not theft of data, since the attacker has no way to see the response to the forged request

**Payloads**

When you are logged into a certain site, you will most likely have a session. The identifier of that session is stored in a cookie in your browser, and is sent with every request to that site. Even if some other site triggers a request, the cookie is sent along with the request and the request is handled as if the logged in user performed it.

HTML GET - No User Interaction
```html
<img src="http://example.com/api/setusername?username=CSRFd">
```

HTML POST - AutoSubmit - No User Interaction
```html
<form id="autosubmit" action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
	  <!-- Below are the values we would put in as if we were in the regular page -->
    <input name="username" type="hidden" value="CSRFd" />
    <input type="submit" value="Submit Request" />
</form>
 
<script>
 document.getElementById("autosubmit").submit();
</script>
```

## Preventing CSRF Attacks

We can prevent CSRF attacks by creating "tokens". Token values must be unique for every person visiting that website. One way we can create tokens is by setting a session variable to random bytes.

Example:

```php
// This creates a random byte that will be unique for every visitor
$_SESSION['token'] = bin2hex(random_bytes(32))
```

Now that we've created our session token, we need to compare this session token to the token you submit via POST request to the page. How does this work?

Lets say we need to click a button, whenever you click that button, you may want to send data over to another script. But this time, to prevent CSRF attacks, you also need to add a hidden field with the value being that of your current CSRF Session token. For example:

```php
<form method="post" action="action-add-thread.php">
	<input type="hidden" name="token" value="<?=$_SESSION['token']?>">
	<textarea id="subject" name="subject" rows="1" cols="80" style="resize: none; margin-left: 20px" placeholder="Subject Title" required></textarea><br>
	<textarea id="message" name="message" rows="8" cols="80" style="resize: none; margin-left: 20px" placeholder="Description of your post" required></textarea><br>
	<input type="submit" value="Add Thread" style="margin-left: 20px"><br>
</form>
```

As we can see, the "value" parameter is equal to that of your session token. Now whenver a user clicks on that button, the token value will be sent along as well. But now we need to actually verify if that token is legit. To do this, we need to compare the hash value of your session token, to the value of the session token you sent when clicking on that button.

Example:

```php
if (hash_equals($_SESSION['token'], $_POST['token'])) {
	// Do stuff
}
```

This compares your current session token to that of the value of which you sent when clicking on that button. 

This prevents CSRF attacks because the attacker won't be able to have the same session token as you do, and if they do happen to grab your POST request token and put it into the website, they still won't be able to do anything because their session is not the same as the one they grabbed.
