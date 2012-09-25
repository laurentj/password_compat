password_compat
===============

This library is intended to provide forward compatibility with the password_* functions being worked on for PHP 5.5.

See [the RFC](https://wiki.php.net/rfc/password_hash) for more detailed information.

Status: BETA-1

Requirements
============

This library requires PHP >= 5.3.3 on Debian Squeeze or 5.3.7 on other distributions. If you attempt to use it on an earlier version, it will raise an error and not define the API.

The reason for this is that PHP prior to 5.3.7 (or prior to 5.3.3-7+squeeze4 on Debian) contains a security issue with its BCRYPT implementation. Therefore, it's highly recommended that you upgrade to a newer version of PHP prior to using this layer.

Installation
============

To install, simply `require` the `password.php` file under `lib`. 

You can also install it via `Composer` by using the [Packagist archive](http://packagist.org/packages/ircmaxell/password-compat).

Usage
=====

**Creating Password Hashes**

To create a password hash from a password, simply use the `password_hash` function.

    $hash = password_hash($password, PASSWORD_BCRYPT);

Note that the algorithm that we chose is `PASSWORD_BCRYPT`. That's the current strongest algorithm supported. This is the `BCRYPT` crypt algorithm. It produces a 60 character hash as the result.

`BCRYPT` also allows for you to define a `cost` parameter in the options array. This allows for you to change the CPU cost of the algorithm:

    $hash = password_hash($password, PASSWORD_BCRYPT, ["cost" => 10]);

That's the same as the default. The cost can range from `4` to `31`. I would suggest that you use the highest cost that you can, while keeping response time reasonable (I target between 0.1 and 0.5 seconds for a hash, depending on use-case).

Another algorithm name is supported:

    PASSWORD_DEFAULT

This will use the strongest algorithm available to PHP at the current time. Presently, this is the same as specifiying `PASSWORD_BCRYPT`. But in future versions of PHP, it may be updated to use a stronger algorithm if one is introduced. It can also be changed if a problem is identified with the BCRYPT algorithm. Note that if you use this option, you are **strongly** encouraged to store it in a `VARCHAR(255)` column to avoid truncation issues if a future algorithm increases the length of the generated hash.

It is very important that you should check the return value of `password_hash` prior to storing it, because a `false` may be returned if it encountered an error.

**Verifying Password Hashes**

To verify a hash created by `password_hash`, simply call:

	if (password_verify($password, $hash)) {
		/* Valid */
	} else {
		/* Invalid */
	}

That's all there is to it.

**Rehashing Passwords**

From time to time you may update your hashing parameters (algorithm, cost, etc). So a function to determine if rehashing is necessary is available:

    if (password_verify($password, $hash)) {
		if (password_needs_rehash($hash, $algorithm, $options)) {
			$hash = password_hash($password, $algorithm, $options);
			/* Store new hash in db */
		}
	}
