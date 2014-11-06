# Silverstripe SSP module #

This module is a wrapper around the [SimpleSAMLphp](http://simplesamlphp.org/)  thirdparty library, providing federated authentication.

It replaces the `Security` controller with the `SSPSecurity` controller that which deals with authenticating users. 

**Caution: This is NOT a plug and play module! This module is just a light wrapper, and requires configuration of the complex SimpleSAMLphp library.**

Parts of this module is based on the [silverstripe-labs/silverstripe-shibboleth](https://github.com/silverstripe-labs/silverstripe-shibboleth) module for Silverstripe 2.4.

## Requirements ##

* SilverStripe 3.1+
* PHP 5.3
* [SimpleSAMLphp requirements](https://simplesamlphp.org/docs/stable/simplesamlphp-install#section_3)

## Installation ##

Use [`Composer`](http://doc.silverstripe.org/framework/en/installation/composer) to install the module in the root folder of your Silverstripe website:
```
composer require antons/silverstripe-ssp
```

### Module setup ###

*Adapt the paths/domains according to your specific environment in the following instructions.*

1. Create a link to the `vendor/simplesamlphp/simplesamlphp/www` folder using any of the following methods:

	a. Create a [mod_alias](http://httpd.apache.org/docs/current/mod/mod_alias.html) directive in the Apache virtual host for your Silverstripe install. Call it `simplesaml` and point it to `vendor/simplesamlphp/simplesamlphp/www` folder. This is the preferred method.
	```
	<VirtualHost *>
		ServerName yourdomain.com
	    DocumentRoot /var/www/ss
		Alias /var/www/ss/simplesaml /var/www/ss/vendor/simplesamlphp/simplesamlphp/www
	</VirtualHost>
	```

	b. Instead of mod_alias, you can create a symbolic link on your system

	```
	//Linux + Mac
	ln -s /var/www/ss/vendor/simplesamlphp/simplesamlphp/www /var/www/ss/simplesaml

	//Windows
	mklink /D C:\xampp\htdocs\ss\simplesaml C:\xampp\htdocs\ss\vendor\simplesamlphp\simplesamlphp\www
	```

	c. If you are using a shared hosting environment and don't have access to the Apache virtual config or the direct filesystem, you can add the following code to your `mysite/_config.php` file and PHP will create the symbolic link.

	```
	if(!file_exists(BASE_PATH . '/simplesaml')) {
    	symlink(realpath(dirname(__FILE__)) . '../vendor/simplesamlphp/simplesamlphp/www', realpath(dirname(__FILE__)) . '/../simplesaml');
	}
	```

2. Insert the following `RewriteCond` before the final `RewriteRule` in the root `.htaccess` file for your Silverstripe website.

	```
	#Add line before 'RewriteRule .* framework/main.php?url=%1&%{QUERY_STRING} [L]'
	RewriteCond %{REQUEST_URI} !simplesaml
	```

### SimpleSAML configuration
1. Make a copy of the default configuration files:
	```
	cd vendor/simplesamlphp/simplesamlphp
	cp -R config-templates config
	cp -R metadata-templates metadata
	```

2. Follow steps 1 to 5 of the [SimpleSAMLphp Service Provider Quickstart](http://simplesamlphp.org/docs/stable/simplesamlphp-sp) documentation to set up SimpleSAMLphp.

### SingleLogoutService (SLO)
If you want to setup the `SingleLogoutService` response location for your identity provider metadata, the URL is:
```
https://yourdomain.com/Security/loggedout
```
This is required for the identity provider to logout properly. 

### SimpleSAMLphp frontend ###
You should be able to access the frontend to SimpleSAMLphp via `https://www.yourdomain.com/simplesaml`. Ensure that you set a password to the frontend in the SimpleSAMLphp config file.


### Adding authenticators to SSPSecurity ###

To add a new authenticator, create a new class that extends `SSPAuthenticator` and implement the `SSPAuthenticator->authenticate()` function:

```
<?php
class MySSPAuthenticator extends SSPAuthenticator {
	public function authenticate() {
		$attributes = $this->getAttributes();

		//Refer to example authenticators in /silverstripe-ssp/code/authenticators

		//SSPSecurity will handle the $member->login() so MySSPAuthenticator->authenticate() 
		//must return a Silverstripe Member object
		
		return $member;
	}
}
```

### config.yml ###

To add authenticators to SSPSecurity:

```
# auth_source is the authentication source defined in SimpleSAMLphp
# auth_class is a SSPAuthentication class that uses auth_source for authentication
# You can add multiple authenticators but at least one authenticator must be specified

SSPSecurity:
  authenticators:
    auth_source: auth_class
```

To set a default authenticator for SSPSecurity:

```
SSPSecurity:
  authenticators:
    auth_source: auth_class
  default_authenticator:
    auth_source

# You can specify default authenticators depending on what the Silverstripe environment mode is
# If an environment is not specified, the first authenticator specified in SSPSecurity::authenticators is used

SSPSecurity:
  authenticators:
    auth_source_1: auth_class_1
    auth_source_2: auth_class_2
    auth_source_3: auth_class_3
  default_authenticator:
    live:
      auth_source_1
    test:
      auth_source_2
    dev:
      auth_source_3
```

### URLs for login/logout ###
`SSPSecurity` replaces the `Security` actions for login/logout:

Login using the default authenticator: 
```
http://yourdomain.com/Security/login
```
Login using an custom authenticator:
```
http://yourdomain.com/Security/login?as=auth_source_1
``` 
Logout: 
```
http://yourdomain.com/Security/logout
```

If a `ContentController` invokes the `Security::PermissionFailure()` function, the user will be redirected to the login page.

## Notes ##
 * The [Security](http://api.silverstripe.org/master/class-Security.html)  class in `silverstripe/framework` is overridden by this module and as a consequence the default Silverstripe login page isn't used. This is by design. If the authentication source uses SimpleSAMLphp for the login page, you can follow the guide here on [theming SimpleSAMLphp](http://simplesamlphp.org/docs/stable/simplesamlphp-theming). If you are using another authentication source with its own login page ie. Google Apps, Microsoft ADFS etc, then you will need to refer to the documentaion of that provider on how to theme it.


## License ##
Copyright (c) 2014, Anton Smith, [newSplash Studio](http://newsplash.co.nz), [Otago Polytechnic](http://www.op.ac.nz)

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

* Neither the name of newSplash Studio nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

