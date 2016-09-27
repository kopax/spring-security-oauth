## About this fork

This fork as been created in order to demonstrate the jdbc token store functionality.

Error are reported at the end.

__Import project in Eclipse__

1. Open eclipse and select `/workspace/github.com/kopax/spring-security-oauth` as a workspace
1. I click on File then `Open projects from File System`.
1. Click on directory then import `/workspace/github.com/kopax/spring-security-oauth`, keep all the Folder checked for import and click Finish, this will load all the project into your workspace.

__Create the database__

You will need to run on your host the following mysql instance :

	hostname: localhost
	port: 3306
	name: oauth2
	user: tutorialuser
	pass: tutorialmy5ql

__Run services__

You will want to run the following app with the following server settings: 

- spring-security-oauth-resource
 - port=8082
 - contextPath=/spring-security-oauth-resource 
- spring-security-oauth-server
 - server.contextPath=/spring-security-oauth-server
 - server.port=8081
- spring-security-oauth-ui-password
 - server.port=8084
 - zuul.routes.oauth.path=/oauth/**
 - zuul.routes.oauth.url=http://localhost:8081/spring-security-oauth-server/oauth

 _Note:_ this are default configuration and should not be changed, just be sure to have 8082,8081,8084 free in order to complete the run.

 1. In eclipse switch to Debug perspective
 1. Click on `Run > Debug configurations`...
 1. Create 3 Java application :
  - resource
   - Project: spring-security-oauth-resource
   - Main class: org.baeldung.config.ResourceServerApplication
  - server
   - Project: spring-security-oauth-server
   - Main class: org.baeldung.config.AuthorizationServerApplication
  - ui-password
   - Project: spring-security-oauth-ui-password
   - Main class: org.baeldung.config.UiApplication
 1. For these 3 Run configuration, go under tab Common and check Debug and Run within `Display in favorites menu`, click Apply.
 1. Run the 3 services

Test services by doing the following command:

Resource: 

    $ curl http://localhost:8082/spring-security-oauth-resource/
    {"error":"unauthorized","error_description":"Full authentication is required to access this resource"}

Authorization:

    $ curl http://localhost:8081/spring-security-oauth-server/oauth/token
    {"timestamp":1474968011125,"status":401,"error":"Unauthorized","message":"Full authentication is required to access this resource","path":"/spring-security-oauth-server/oauth/token"}

Client angular:

    $ curl -X POST http://localhost:8084/oauth/token
    {"error":"invalid_request","error_description":"Missing grant type"}

Database:

	$ mysql -h localhost -u tutorialuser -ptutorialmy5ql -P3306 -D oauth2 -e "show tables;"
	+----------------------+
	| Tables_in_DATA_V1    |
	+----------------------+
	| ClientDetails        |
	| oauth_access_token   |
	| oauth_approvals      |
	| oauth_client_details |
	| oauth_client_token   |
	| oauth_code           |
	| oauth_refresh_token  |
	+----------------------+

You now have validated the environment, let's do the testing:

Start Google chrome and go on the client interface:

1. Go into `Incognito` in order to have a clean cookie and cache sandbox.
1. http://localhost:8084 , you should see the login form
1. Press F12 to open the developer tools, go under Network tab and click on `Preserve log` and check `XHR filter`
1. Now refresh the view by pressing F5
1. Under the Resources tab in the developer tools, verify the non presence of cookie for localhost, if you do have, please restart chrome and go back into Incognito mode then try again.
1. By doing refresh, under the Network tab, you will notice the login POST request automatically made to http://localhost:8084/oauth/token with a Status Code 400 OK
1. Try to login using the inMemory credentials of your choice : 
  - tom:111 (ADMIN)
1. Press login, you will now see a second POST request to http://localhost:8084/oauth/token containing the form data ({ client_id:fooClientIdPassword, grant_type:password, password:111, username:tom }), status code is 200 and you have a Set-Cookie in the response Header `Set-Cookie:refreshToken=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJ0b20iLCJzY29wZSI6WyJmb28iLCJyZWFkIiwid3JpdGUiXSwib3JnYW5pemF0aW9uIjoidG9tV3lIYSIsImF0aSI6IjA5NjYyNGMzLTE4NzAtNGM2My05YTg4LTg0ODAwZGIzMDc3MiIsImV4cCI6MTQ3NzU2MDk1MCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJlMGE1...1lMjA2MGFjMDNiYjMiLCJjbGllbnRfaWQiOiJmb29DbGllbnRJZFBhc3N3b3JkIn0.O0XUHm7ilf3ECg3AJS9ftMoLhhst_xbPOm8T_6GOTClF_Rw85qyXzE-6mYEMYrqIUMpMCFTsdNFbpRWolW9SBJLrwRuvTdMqtgAZstn0nQzmedJDBoFtJF5QcFgLuPqhn9yP_orVzGHwo_td_62w7XiFwz_4_tSeoXsSjs1Z_uoC_SXmNJ1VFG-m_TYgP_y6gLXYmmXrVQz7WMu9um0A708UoQc7UKJyrFXrlb7VLFq7Ni1yQy7uI9l2DjXGGiTKLzrsthQmAiBFi3VGp-Gozbf8a9D2huUq7H6pjQupX-kaQXZFs44n_rUZxGKdhtMZCGOkJzBL_Q-T_AayOtJg9w;Max-Age=2592000;path=/oauth/token;HttpOnly`
1. Under your resources tab, you will now see the cookie : `access_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJ0b20iLCJzY29wZSI6WyJmb28iLCJyZWFkIiwid3JpdGUiXSwib3JnYW5pemF0aW9uIjoidG9tV3lIYSIsImV4cCI6MTQ3NDk3MjU1MCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiIwOTY2MjRjMy0xODcwLTRjNjMtOWE4OC04NDgwMGRiMzA3NzIiL...lbnRfaWQiOiJmb29DbGllbnRJZFBhc3N3b3JkIn0.EDpo9nWEv9Ir_IJZ7fDkMRI59TVcfuKaaaMf1tu6zxuY-c4r_KAHXhU1gzCbTDsUoNFrtq9CHRzS9zNpB_RYbuboWEhTrPdL4qv01rH7dKYhQvrvYs6qQ9oxOtbSdR4O_96oCobnzIRVSLLX4cjUxPbRUjkKh7i5wYSDa2LMRF5hXO9_7WQA9bSWS0vYF2c35pCzXSkyw9PSssosz_zcy4YMCuMFA06RraRVfjZnLDYkYk0m23VjXG8vAUy7af_rcr410g4s_tbeK1NiyhO8icN_x_jDD9crjj7yp41J1D8zAj4maBsQothbhAtDlqR9RRJyVwlDGtgG-PjikeeRhQ`
1. In bash, verify the presence of the access_token:  

	$ mysql -h localhost -u tutorialuser -ptutorialmy5ql -P3306 -D oauth2 -e "select user_name, refresh_token from oauth_access_token"
		+-----------+----------------------------------+
		| user_name | refresh_token                    |
		+-----------+----------------------------------+
		| tom       | f70d26129315697031...658bd40e6a4 |
		+-----------+----------------------------------+
	$ mysql -h localhost -u tutorialuser -ptutorialmy5ql -P3306 -D oauth2 -e "select token_id from oauth_refresh_token"
		+----------------------------------+
		| token_id                         |
		+----------------------------------+
		| 4c5ce06fae38f09f5d9796314d46971b |
		+----------------------------------+
1. From now on, we can think everything is working fine, we can't logout so we will just remove the cookie and restart again, under the Resources tab, remove all cookie for localhost then press F5, you will be on login.
1. Try to login using `tom:111`, you will now see a POST to http://localhost:8084/oauth/token Status code 400
1. If you go and check your server log in eclipse, you will have the following error : 

	2016-09-27 16:44:23.429  INFO 5552 --- [nio-8081-exec-3] o.s.s.o.p.token.store.JdbcTokenStore     : Failed to find refresh token for token eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJ0b20iLCJzY29wZSI6WyJmb28iLCJyZWFkIiwid3JpdGUiXSwib3JnYW5pemF0aW9uIjoidG9tekdOSyIsImF0aSI6IjVkZWVhY2RlLTIzYTEtNGQxMy1iM2JjLTM5OWFhYzBlZWZjMiIsImV4cCI6MTQ3NzU2MDk1MCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJlMGE1ZDk5MS0yY2YyLTRlOGItOGZiN...jA2MGFjMDNiYjMiLCJjbGllbnRfaWQiOiJmb29DbGllbnRJZFBhc3N3b3JkIn0.c1rRT3DMjGhSOyvp6e6trCVp1TbBt_RFdco11KPbcG9OUjdN1sq_xnH0qOwGlUMx40pXwkCIS_NhnH_asvMz-5jKyXrYiUDNE5xyJww_DJctFNALyVxNo0Kjf5TDElmFLoXiUpLz_iUKbpKEIZqf1GmrXkJfVgFGk1J9ast6IoZGBPvH3lCxpHYKnzCjhTbPA-wCpzIrR_idVOATylwoCUknO6HxqNgnsH1YdlQ2J_2ZyxDP2qq_SNKOT9wqQ9zJpI7OjBZ-Woy_DRQrgpwdmSft5luDWIWcrraZ8eGsK1YgHUBxV1Fy1NUlFmvELF42ZAxOiXCbVTZftTdBXzltXw
	2016-09-27 16:44:23.433  INFO 5552 --- [nio-8081-exec-3] o.s.s.o.provider.endpoint.TokenEndpoint  : Handling error: InvalidGrantException, Invalid refresh token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJ0b20iLCJzY29wZSI6WyJmb28iLCJyZWFkIiwid3JpdGUiXSwib3JnYW5pemF0aW9uIjoidG9tekdOSyIsImF0aSI6IjVkZWVhY2RlLTIzYTEtNGQxMy1iM2JjLTM5OWFhYzBlZWZjMiIsImV4cCI6MTQ3NzU2MDk1MCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiO...GE1ZDk5MS0yY2YyLTRlOGItOGZiNy1lMjA2MGFjMDNiYjMiLCJjbGllbnRfaWQiOiJmb29DbGllbnRJZFBhc3N3b3JkIn0.c1rRT3DMjGhSOyvp6e6trCVp1TbBt_RFdco11KPbcG9OUjdN1sq_xnH0qOwGlUMx40pXwkCIS_NhnH_asvMz-5jKyXrYiUDNE5xyJww_DJctFNALyVxNo0Kjf5TDElmFLoXiUpLz_iUKbpKEIZqf1GmrXkJfVgFGk1J9ast6IoZGBPvH3lCxpHYKnzCjhTbPA-wCpzIrR_idVOATylwoCUknO6HxqNgnsH1YdlQ2J_2ZyxDP2qq_SNKOT9wqQ9zJpI7OjBZ-Woy_DRQrgpwdmSft5luDWIWcrraZ8eGsK1YgHUBxV1Fy1NUlFmvELF42ZAxOiXCbVTZftTdBXzltXw

1. Try to login with john:123, same problem, can't login, error 400.

## Spring Security OAuth

### Relevant Articles: 
- [Spring REST API + OAuth2 + AngularJS](http://www.baeldung.com/rest-api-spring-oauth2-angularjs)

### Build the Project
```
mvn clean install
```

### Notes
- This project consists of 4 main sub-modules, each sub-module is a Spring Boot Application running on specific port
    - spring-security-oauth-server       (port = 8081)
    - spring-security-oauth-resource     (port = 8082)
    - spring-security-oauth-ui-implicit  (port = 8083)
    - spring-security-oauth-ui-password  (port = 8084)
- To run the project, run both _spring-security-oauth-server_ and _spring-security-oauth-resource_ first - then run any of the UI modules

- You can run any sub-module using command line: 
```
mvn spring-boot:run
```

- _spring-security-oauth-ui-password_ has "live" profile for live tests, in order to run it use the following command: 
```
mvn clean install -Plive
```
