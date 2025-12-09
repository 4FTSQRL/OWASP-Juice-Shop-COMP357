Lab Creation-Juice Shop 

 

Basic Topology Diagram 

 

I was running this through docker on a Windows 11 Machine. Docker was already set up on my machine. 

 Application Deployment 

I pulled a docker instance from bkimminich/juice-shop. 

I ran the docker on the localhost using port 3000. 

 

Attack Report 

SQL Injection 

Target 

The first thing that stuck out to me was the account log in. I wanted to try to get into the Administrator’s account with a SQL injection. 

Exploit 

 

I tried this SQL injection. It is ending the username selection and is asking if 1 is equal to 1. It checks for the correct username or if 1=1. 

I was able to successfully log in. However, I was curious about other SQL injections that were available. 

 

I logged back into the administrator using the previous injection. 

 

I tried entering the SQL injection above. This SQL injection looks for “anything” or if 1 is equal to 1 in both the username and password field. While it did not allow me to log in, it did present me with another exploit. I triggered an error that the application could not handle. 

Purple Team Mitigation Report 

Sanitization is the most popular way to mitigate against SQL injections.  

In /juice-shop/routes/login.ts in the Docker container juice-shop-waf-juice-shop-1, I found some of the insecure code. 

The database was being searched with the following line: 

    models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true }) // vuln-code-snippet vuln-line loginAdminChallenge loginBenderChallenge loginJimChallenge 

 

I adjusted it to handle replacements rather than plaintext. 

    models.sequelize.query(`SELECT * FROM Users WHERE email = :email AND password = :password AND deletedAt IS NULL`, {replacements: { email: email, password: password }, model: UserModel, plain: true }); // vuln-code-snippet vuln-line loginAdminChallenge loginBenderChallenge loginJimChallenge 

 

Unfortunately, the exploit was still successful.  
