---
title: "Access Okta APIs through httpie and jq"
date: 2022-10-11T16:26:25+05:30
draft: false
image: ""
tags: [okta,api]
---

**Table of Contents**

1. [Pre-requisite](#pre-requisite)
2. [Access Okta APIs through HTTPie](#access-okta-apis-through-httpie)
3. [Using jq to process JSON response](#using-jq-to-process-json-response)
4. [Access multiple Okta tenants using sessions in HTTPie](#access-multiple-okta-tenants-using-sessions-in-httpie)


----


Most of us use [Postman](https://www.postman.com/product/what-is-postman/) as the tool to access APIs.

As someone who likes cli(command line interface), when I was exploring HTTP clients I found **HTTPie** 
(_pronounced as **aitch-tee-tee-pie**_) and **Jq**. Jq is a JSON processor that allows user to retrieve certain records or name-value pairs from it.

I found it difficult to view specific attributes for a user or group and compare it with another user while using the Postman.

Below shown is one user record that is returned when a HTTP GET request is made to `/users` endpoint using API. If I need the values for _First Name_,_Last Name_ and _Date of Activation_, I have to search for the values in the response.


```
    {
        "id": "00uboac18l2NDtseS4x7",
        "status": "PROVISIONED",
        "created": "2022-03-08T10:59:59.000Z",
        "activated": "2022-03-08T10:59:59.000Z",
        "statusChanged": "2022-03-08T10:59:59.000Z",
        "lastLogin": null,
        "lastUpdated": "2022-03-08T10:59:59.000Z",
        "passwordChanged": null,
        "type": {
            "id": "otyc373d9y5oyWY174x6"
        },
        "profile": {
            "firstName": "Test",
            "lastName": "User002",
            "mobilePhone": null,
            "secondEmail": null,
            "login": "testuser002@corp.com",
            "email": "testuser002@corp.com"
        },
        "credentials": {
            "emails": [
                {
                    "value": "testuser002@corp.com",
                    "status": "VERIFIED",
                    "type": "PRIMARY"
                }
            ],
            "provider": {
                "type": "OKTA",
                "name": "OKTA"
            }
        },
        "_links": {
            "self": {
                "href": "https://dev-12345.okta.com/api/v1/users/00uboac18l2NDtseS4x7"
            }
        }
    },

```


-----

## Pre-requisite

1. API token created on your Okta tenant. You can follow the [Okta documentation to create an API token](https://developer.okta.com/docs/guides/create-an-api-token/main/). For this blog post I created an API token on my developer tenant.

2. Install httpie by following the instructions in the [official documentation](https://httpie.io/docs/cli/installation). Httpie can be installed on Windows,Linux and MacOS.

3. Install jq by following the instructions in the [documentation](https://jqlang.github.io/jq/download/). Jq can be installed on Windows,Linux and MacOS.

4. To verify if both **httpie** and **jq** are installed , issue the below command. A response with "hello world" in JSON format should be received as shown 

    `http https://httpbin.org/get?message="hello world" | jq .`



``` 
{
  "args": {
    "message": "hello world"
  },
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Host": "httpbin.org",
    "User-Agent": "HTTPie/3.2.1",
    "X-Amzn-Trace-Id": "Root=1-64eb6355-19c0a1b82a9fda080833fc7f"
  },
  "origin": "123.123.123.98",
  "url": "https://httpbin.org/get?message=hello world"
}


```

----

## Access Okta APIs through HTTPie

1. The Okta API uses the custom HTTP authentication scheme `SSWS` for API authentication. An API request must have valid API token prefixed with `SSWS` in the HTTP `Authorization` header.

    **Authorization: SSWS {API token here}**

    `Authorization: SSWS 00ABjCD4MeF-BDXM...ABmjXF-vbGua`

2. To get the list of users, we can access `/users` API endpoint through httpie as shown here.

    `http https://dev-12345.okta.com/api/v1/users Authorization:'SSWS 00ABjCD4MeF-BDXM...ABmjXF-vbGua'`

3. Headers can also be read from a file using the `:@` operator while using httpie. We can get the list of users by

    - Create a file with name `Headers.txt` which contains the value for `Authorization` header

    ```
    cat headers.txt 
    SSWS 00ABjCD4MeF-BDXM...ABmjXF-vbGua

    ```

    - Provide reference to the file `headers.txt` in the API request  through `http` command.

    `http https://dev-12345.okta.com/api/v1/users Authorization:@headers.txt`


----

## Using jq to process JSON response

1. The best practice using `jq` with `http` is to pass the output of the API response to jq.

    `http https://dev-12345.okta.com/api/v1/users Authorization:@headers.txt | jq .`

2. If the response contains an array, the values can be iterated. The jq documentation on [devdocs.io](https://devdocs.io/jq/) provides good examples.

3. When I required the values of _First Name_,_Last Name_ and _Date of Activation_ , I first verified the JSON response of the user object.

4. Below  is a visual representation of the response from Okta for a user shown earlier in the form of JSON.


    ![user-object-json-diagram](/images/2022/10/user-object-json-response.png)


4. So from the JSON response, jq will have to filter `activated`, `firstName` and `lastName` under `profile`

5. The command will look like 

    `http -b GET https://dev-12345.okta.com/api/v1/users Authorization:@headers.txt  | jq '.[] | { "Date of Activation": .activated , "First Name": .profile.firstName , "Last Name": .profile.lastName }'`

6. The output of the jq command will be just three values for all the users. Please note that the number of records in the response depends on the limit set by Okta or specified in the command. My developer tenant has only 4 users, the output contains values for all the 4 users.


```
{
  "Date of Activation": "2022-03-08T10:59:59.000Z",
  "First Name": "Test",
  "Last Name": "User002"
}
{
  "Date of Activation": "2022-06-23T09:47:17.000Z",
  "First Name": "Ramesh",
  "Last Name": "RV"
}
{
  "Date of Activation": "2022-06-23T09:56:30.000Z",
  "First Name": "Okta Super",
  "Last Name": "Admin"
}
{
  "Date of Activation": "2020-05-15T00:45:10.000Z",
  "First Name": "AD",
  "Last Name": "Admin"
}


```

7. `jq` will save time required to view response for each user and search for the values of _First Name_,_Last Name_ and _Date of Activation_.

8. We can also use HTTPie to create users,groups or applications in Okta. I will cover them in another post.

----

## Access multiple Okta tenants using sessions in HTTPie

- HTTPie also supports persistent sessions via the `--session=SESSION_NAME_or_PATH_TO_SESSION` option.

- If we have two different Okta tenants like `dev-12345.okta.com` and `dev-54321.okta.com`, we can use the sessions option to access data from both the environments with reduced effort.

- Create a directory and provide the name of the tenant. Example: dev-12345.okta.com.

- Inside the directory create a file called `config.json` and provide the Authorization header and API key as shown below.

    ```
    {

    "headers": [
            {
                "name": "Authorization",
                "value": "SSWS enter the API key here"
            }
        ]
    }
  
    ```
- Create the directory and repeat the process for the other tenant dev-54321.okta.com. The API key will be unique for each tenant.

- The directory structure can be similar to the output shown below

    ```
        /home/ramesh/.config/httpie/sessions/
    ├── dev-12345.okta.com
    │   └── config.json
    └── dev-54321.okta.com
        └── config.json

    3 directories, 2 files

    
    ```
- Configure two environment variables `OKTA_HOST` and `SESSION_HOME`

    ``` bash
    export OKTA_HOST=dev-12345.okta.com
    export SESSION_HOME=~/.config/httpie/sessions/$OKTA_HOST

    ```
- To retrieve the values _First Name_,_Last Name_ and _Date of Activation_ from `dev-12345.okta.com` we can execute the `http` command as shown below

   ```
   http -b GET https://$OKTA_HOST/api/v1/users --session-read-only=$SESSION_HOME/config.json | jq '.[] | { "Date of Activation": .activated , "First Name": .profile.firstName , "Last Name": .profile.lastName }'

   ```
- To retrieve the values for _First Name_,_Last Name_ and _Date of Activation_ from `dev-54321.okta.com` , we can modify only the `OKTA_HOST` environment variable and execute the above shown `http` command

    ``` bash
    export OKTA_HOST=dev-54321.okta.com

    ```
- This can be helpful if we have to compare the values from different Okta tenants.

----
