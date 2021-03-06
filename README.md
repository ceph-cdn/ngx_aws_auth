# AWS proxy module

[![Build Status](https://travis-ci.org/anomalizer/ngx_aws_auth.svg?branch=master)](https://travis-ci.org/anomalizer/ngx_aws_auth)
 [![Gitter chat](https://badges.gitter.im/anomalizer/ngx_aws_auth.png)](https://gitter.im/ngx_aws_auth/Lobby?utm_source=share-link&utm_medium=link&utm_campaign=share-link)

This nginx module can proxy requests to authenticated S3 backends using Amazon's
V4 authentication API. The first version of this module was written for the V2
authentication protocol and can be found in the *AuthV2* branch.
The module is compatible with Ceph luminous.

## License
This project uses the same license as ngnix does i.e. the 2 clause BSD / simplified BSD / FreeBSD license

## Usage example

Implements proxying of authenticated requests to S3.

```nginx
  server {
    listen     8000;

    aws_access_key your_aws_access_key; # Example AKIDEXAMPLE
    aws_key_scope scope_of_generated_signing_key; #Example 20150830/us-east-1/service/aws4_request
    aws_signing_key signing_key_generated_using_script; #Example L4vRLWAO92X5L3Sqk5QydUSdB0nC9+1wfqLMOKLbRp4=
	aws_s3_bucket your_s3_bucket;
	
    upstream buckets {
        keepalive 60; # anable http keepalive support with the s3 servers
        server bucket1.s3.somedomain.ru weight=1 fail_timeout=1s;
        server bucket2.s3.somedomain.ru weight=1 fail_timeout=1s;
    }

    # This is an example that use specific s3 endpoint, default endpoint is s3.amazonaws.com
    location /s3_beijing {
      client_max_body_size 100m;
      client_body_buffer_size 1m;
	
      proxy_http_version 1.1;
      proxy_set_header Connection "";                   # anable keep-alive support with client
      proxy_pass http://buckets;
      proxy_set_header Host real-host;                  # real-host = <bucket>.s3domain. S3 server will use this value to check the signature

      aws_sign;
      aws_version "v4";                                 # Default v4. Could be set into v2 or v4. v2 signature version hasn't ready yet
      aws_endpoint "s3domain";                          # without bucket name
      aws_s3_bucket "bucketname";                       # your bucket name
      aws_access_key your_aws_access_key;
      aws_signing_key signing_key_generated_using_script;
      aws_key_scope region/service/aws4_request;        # For example: "us-east-1/s3/aws4_request". The current date will be set automatically
    }
  }
```

## Security considerations
The V4 protocol does not need access to the actual secret keys that one obtains 
from the IAM service. The correct way to use the IAM key is to actually generate
a scoped signing key and use this signing key to access S3. This nginx module
requires the signing key and not the actual secret key. It is an insecure practise
to let the secret key reside on your nginx server.

Note that signing keys have a validity of just one week. Hence, they need to
be refreshed constantly. Please useyour favourite configuration management
system such as saltstack, puppet, chef, etc. etc. to distribute the signing
keys to your nginx clusters. Do not forget to HUP the server after placing the new
signing key as nginx reads the configuration only at startup time.

## Known limitations
The 2.x version of the module hasn't support POST multipart form data yet. Use PUT HTTP method to create objects


## Credits
Original idea based on http://nginx.org/pipermail/nginx/2010-February/018583.html and suggestion of moving to variables rather than patching the proxy module.

Subsequent contributions can be found in the commit logs of the project.
