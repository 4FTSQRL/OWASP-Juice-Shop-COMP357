# Lab Creation-Juice Shop 

I was running this through docker on a Windows 11 Machine. Docker was already set up on my machine. 

<h1>Application Deployment </h1>

I pulled a docker instance from bkimminich/juice-shop. 
<pre>docker pull bkimminich/juice-shop</pre>

I ran the docker on the localhost using port 3000. 
<pre>docker run --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop</pre>
 

<h1>Attack Report</h1>

<h2>SQL Injection</h2>

<h3>Target</h3>

The first thing that stuck out to me was the account log in. I wanted to try to get into the Administrator’s account with a SQL injection. 

<h3>Exploit</h3>

<img width="322" height="288" alt="image" src="https://github.com/user-attachments/assets/269f77e8-7634-47af-8535-f47911e8cff6" />

I tried this SQL injection. It is ending the username selection and is asking if 1 is equal to 1. It checks for the correct username or if 1=1. 
<img width="975" height="278" alt="image" src="https://github.com/user-attachments/assets/f8ead373-f68d-49d8-8d19-58273205c2bb" />

I was able to successfully log in. However, I was curious about other SQL injections that were available. 

 

I logged back into the administrator using the previous injection. 

<img width="717" height="644" alt="image" src="https://github.com/user-attachments/assets/59693b77-29ad-4a19-af04-2105ebf10806" />


I tried entering the SQL injection above. This SQL injection looks for “anything” or if 1 is equal to 1 in both the username and password field. While it did not allow me to log in, it did present me with another exploit. I triggered an error that the application could not handle. 

<h1>Purple Team Mitigation Report</h1> 

Sanitization is the most popular way to mitigate against SQL injections.  

In /juice-shop/routes/login.ts in the Docker container juice-shop-waf-juice-shop-1, I found some of the insecure code. 

The database was being searched with the following line: 

    models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true }) // vuln-code-snippet vuln-line loginAdminChallenge loginBenderChallenge loginJimChallenge 

 

I adjusted it to handle replacements rather than plaintext. 

    models.sequelize.query(`SELECT * FROM Users WHERE email = :email AND password = :password AND deletedAt IS NULL`, {replacements: { email: email, password: password }, model: UserModel, plain: true }); // vuln-code-snippet vuln-line loginAdminChallenge loginBenderChallenge loginJimChallenge 

 

Unfortunately, the exploit was still successful.  
