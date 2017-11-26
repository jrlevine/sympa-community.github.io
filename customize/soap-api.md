SOAP/HTTP API
=============

Introduction
------------

(Work in progress)

Supported functions
-------------------

Note that all functions accessible through the SOAP interface apply the appropriate access control rules, given the user's privileges.

The following functions are currently available through the Sympa SOAP server :

  - `login`

    User email and passwords are checked against Sympa user DB, or another backend.

  - `casLogin`

    This function will verify CAS proxy tickets against the CAS server.

  - `authenticateAndRun`

    Useful for SOAP clients that can't set an HTTP cookie ; they can provide both the Sympa session cookie and the requested command in a single call.

  - `authenticateRemoteAppAndRun`

    Equivalent of the previous command used in a trusted context (see [Trust remote applications](#trust-remote-applications)).

  - `lists`

    Provides a list of available lists (authorization scenarios are applied).

  - `complexLists`

    Same as the previous feature, but provides a complex structure for each list.

  - `info`

    Provides description informations about a given list.

  - `which`

    Gets the list of subscription of a given user.

  - `complexWhich`

    Same as previous command, but provides a complex structure for each list.

  - `amI`

    Tells if a given user is member of a given list.

  - `review`

    Lists the members of a given list.

  - `subscribe`

    Subscribes the current user to a given list.

  - `signoff`

    Current user is removed from a given list.

  - `add`

    Used to add a given user to a given list (admin feature).

  - `del`

    Removes a given user from a given list (admin feature).

  - `createList`

    Creates a new mailing list (requires appropriate privileges).

  - `closeList`

    Closes a given mailing list (admin feature).

Note that when a list parameter is required for a function, you can either provide the list name or the list address. However the domain part of the address will be ignored.

Check [the WSDL service description](../man/sympa.wsdl.5.md) for detailed API information.

Setup
-----

### Requirements

  * Web interface has to be configured. See
    "[Configure HTTP server](../install/configure-http-server.md)" for
    details.

### HTTP server setup

Here are excerts of HTTP server configuration with a SOAP server (Note:
replace [``$EXECCGIDIR``](../layout.md#execcgidir) below).
For more details see appropriate section in
"[Configure HTTP server](../install/configure-http-server.md)".

  * Apache HTTP Server

    ``` code
    <Location /sympasoap>
        SetHandler fcgid-script
    
        # Don't forget to edit lines below!
        Require all denied
        #Require all granted
    </Location>
    ScriptAlias /sympasoap $EXECCGIDIR/sympa_soap_server-wrapper.fcgi
    ```

    ----
    Note:

      * Starting Sympa 5.4, the
        [`sympa_soap_server.fcgi`](../man/sympa_soap_server.8.md) is wrapped
        in small C program, [`sympa_soap_server-wrapper.fcgi`](../man/sympa_soap_server-wrapper.8.md),
        in order to avoid to use the --- insecure and no longer
        maintained --- setuid perl mode.

    ----

### Sympa configuration parameters

  * [``soap_url``](../man/sympa.conf.5.md#soap_url)

    The URL of SOAP/HTTP service itself.

  * [``wwsympa_url``](../man/sympa.conf.5.md#soap_url)

    This is URL prefix of WWSympa service _without_ trailing slash (``/``).
    WSDL service description is published using a URL _wwsymp_url_`/wsdl`.

### Other configuration files

  * [``sympa.wsdl``](../man/sympa.wsdl.5.md)

    WSDL service description. Default description is placed under
    [``$DEFAULTDIR``](../layout.md#defaultdir).

  * [``trusted_applications.conf``](../man/trusted_applications.5.md)

    Definitions of trusted SOAP applications.  See also below.

Trust remote applications
-------------------------

(Work in progress)

Below is a sample Perl code that does a SOAP procedure call (for a SUBSCRIBE sympa command) using the trusted\_application feature :

``` perl
use SOAP::Lite;

my $soap = SOAP::Lite->new();
$soap->uri('urn:sympasoap');
$soap->proxy('http://web.exemple.org/sympasoap');

my $response = $soap->authenticateRemoteAppAndRun('myTestApp',
    'myTestAppPwd', 'USER_EMAIL=userProxy@my.server',
    'subscribe', ['myList@mail.example.org']);
```

[S. Santoro](mailto:dereckson@espace-win.org) wrote its own [PHP Trusted Application library for Sympa](/contribs/index#php_soap_library).

Client-side programming
-----------------------

(Work in progress)

### Writing a Java client with Axis

(Work in progress)

The test command line SOAP client
---------------------------------

Sympa distribution includes a simple command line application that allows you to test SOAP request towards your Sympa SOAP server. This script is named [``sympa_soap_client.pl``](../man/sympa_soap_client.1.md) and is located in [``$SCRIPTDIR``](../layout.md#scriptdir) directory.

The ~~[four methods](/manual/soap#introduction)~~ available through the Sympa SOAP server can be tested using this tool. There is no explicit option to tell what acces methos is used. It is inferred based on what options are provided to the script.

### Getting the email associated to a session id

You must use the id of a session actually used at the time you launch the command. It is the value of the "`sympa_session`" cookie set when accessing to the HTTP interface.

#### Command line

``` bash
$ sympa_soap_client.pl \
 --soap_url=<SOAP server URL> \
 --cookie=<cookie identifier>
```

  - ``--soap_url``

    The URL to your Sympa SOAP server.

  - ``--cookie``

    The value of the "`sympa_session`" cookie set when accessing to the Sympa web interface.

#### Expected output

``` code
error : get_email_cookie
cookie : 65354224256806


getEmailUserByCookie....
0
        'mail@renater.fr'
```

### Using the Sympa SOAP functions with the command line tool

It is done by calling the script and providing two kind of arguments :

  - the argument required by the service usage : SOAP URL, service name and service parameters,

  - the arguments allowing to authenticate the user requesting the service.

#### Authentication using an HTTP session cookie

Actually, providing the HTTP cookie to a command line sums up in providing a session id, i.e. a simple number. You must use the value of a session cookie actually used at the time you launch the command. It is the “sympa\_session” cookie set when accessing to the Sympa web interface.

``` bash
$ sympa_soap_client.pl --soap_url=<SOAP server URL> \
 --service=<a sympa service> \
 --service_parameters=<value1,value2,value3> \
 --session_id=<cookie identifier>
```

The options used are:

  - ``--soap\_url``

    The URL to your Sympa SOAP server.

  - ``--service``

    The requested SOAP service. See [below](#sympa-soap-services-and-the-command-line-tool);

  - ``--service_parameters``

    The parameters needed to use the service. They must be provided as a comma separated list, without spaces. See [below](#sympa-soap-services-and-the-command-line-tool).

  - ``--session_id``

    The value of the "`sympa_session`" cookie set when accessing to the Sympa web interface.

#### Authentication using a user name and password

``` bash
$ sympa_soap_client.pl --soap_url=<SOAP server URL> \
 --service=<a sympa service> \
 --service_parameters=<value1,value2,value3> \
 --user_email=<email> \
 --user_password=<password>
```

The options used are:

  - ``--soap_url``

    The URL to your Sympa SOAP server.

  - ``--service``

    The requested SOAP service. See [below](#sympa-soap-services-and-the-command-line-tool).

  - ``--service_parameters``

    The parameters needed to use the service. They must be provided as a comma separated list, without spaces. See [below](#sympa-soap-services-and-the-command-line-tool).

  - ``--user_email``

    The email of the user requesting the service.

  - ``--user_password``

    The password of this user.

#### Access through a trusted application

``` bash
$ sympa_soap_client.pl --soap_url=<SOAP server URL> \
 --service=<a sympa service> \
 --service_parameters=<value1,value2,value3> \
 --cookie=<cookie identifier> \
 --trusted_application=<app name> \
 --trusted_application_password=<password> \
 --proxy_vars=<id=value,id2=value2>
```

The options used are:

  - ``--soap_url``

    The URL to your Sympa SOAP server.

  - ``--service``

    The requested SOAP service. See [below](#sympa-soap-services-and-the-command-line-tool).

  - ``--service_parameters``

    The parameters needed to use the service. They must be provided as a comma separated list, without spaces. See [below](#sympa-soap-services-and-the-command-line-tool).

  - ``--cookie``

    The value of the "`sympa_session`" cookie set when accessing to the Sympa web interface.

  - ``--trusted_application``

    The trusted application name as defined in `trusted_applications.conf`.

  - ``--trusted_application_password``

    The password of the trusted application as defined in `trusted_applications.conf`.

  - ``--proxy_vars``

    The proxy vars of the trusted application as defined in `trusted_applications.conf`. This is a comma-separated list of values. For example, if you have defined in `trusted_applications.conf` the following variables: `proxy_for_variables USER_EMAIL,remote_host`, then you will use it this way in the proxy\_vars option: `--proxy_vars=USER_EMAIL=user.email@domain.tld,remote_host=remote.host.domain.tld`.

### Sympa SOAP services and the command line tool

This is a description of how to use the Sympa SOAP services using the command line tool. The parameters are given in the same order they must be found in the command tool option `service_parameters`. They must be provided as a comma separated list, without spaces. Don't forget to escape characters that would break the command line, such as spaces, exclamation marks and so on.

----
Note:

  * If the list of parameters is:

      - list name

      - user email

    then the `service_parameters` option will look like:

    ```
    --service_parameters=mylist,mail@my.dom.ain
    ```

----

#### login

No object here: this is the service used to log when the command tool uses a username and password.

#### casLogin

No object here.

#### authenticateAndRun

No object here: this the service used by the command line tool to call the other services, when authentication is done through session id or user name + password.

#### authenticateRemoteAppAndRun

No object here: this the service used by the command line tool to call the other services, when testing trusted applications.

#### lists

The parameters are optional.

Parameters:

  - topic: the topic of the lists to return

  - subtopic: the subtopic of this topic

Output example:

``` code
lists....
0
        'homepage=http://domain.tld/sympa/info/amietestdv01;subject=Amical;listAddress=amietestdv01@domain.tld'
1
        'homepage=http://domain.tld/sympa/info/archeologie;subject=Liste sur l'archéologie;listAddress=archeologie@domain.tld'
2
        'homepage=http://domain.tld/sympa/info/blackmambo;subject=A black mambo;listAddress=blackmambo@domain.tld'
3
        'homepage=http://domain.tld/sympa/info/bluemambo;subject=Another mambo. This one is blue.;listAddress=bluemambo@domain.tld'
```

#### complexLists

The parameters are optional.

Parameters:

  - topic: the topic of the lists to return

  - subtopic: the subtopic of this topic

Output example:

``` code
AuthenticateAndRun complexLists....
0
        _homepage_
                'http://domain.tld/sympa-dv/info/amietestdv01'
        _listAddress_
                'amietestdv01@domain.tld'
        _subject_
                'Amical'
1
        _homepage_
                'http://domain.tld/sympa-dv/info/archeologie'
        _listAddress_
                'archeologie@domain.tld'
        _subject_
                'List sur l'archéologie'
2
        _homepage_
                'http://domain.tld/sympa-dv/info/blackmambo'
        _listAddress_
                'blackmambo@domain.tld'
        _subject_
                'A black mambo'
3
        _homepage_
                'http://domain.tld/sympa-dv/info/bluemambo'
        _listAddress_
                'bluemambo@domain.tld'
        _subject_
                'Another mambo. This one is blue.'
```

#### info

Parameters:

  - listname (mandatory): the name of the list for which info are requested

Output example:

``` code
```

#### which

All arguments are mandatory (at least with an empty value).

Parameters:

  - no parameters

Output example:

``` code
which....
0
        'isOwner=1;homepage=http://domain.tld/sympa/info/amietestdv01;subject=Amical;listAddress=amietestdv01@domain.tld;isEditor=0;isSubscriber=0'
1
        'isOwner=1;homepage=http://domain.tld/sympa/info/archeologie;subject=Liste sur l'archéologie;listAddress=archeologie@domain.tld;isEditor=0;isSubscriber=0'
2
        'isOwner=1;homepage=http://domain.tld/sympa/info/blackmambo;subject=A black mambo;listAddress=blackmambo@domain.tld;isEditor=0;isSubscriber=0'
3
        'isOwner=1;homepage=http://domain.tld/sympa/info/bluemambo;subject=Another mambo. This one is blue.;listAddress=bluemambo@domain.tld;isEditor=0;isSubscriber=0'
```

#### complexWhich

All arguments are mandatory (at least with an empty value).

Parameters:

  - no parameters

Output example:

``` code
complexWhich....
0
        _homepage_
                'http://dev-sympa.renater.fr/sympa-dv/info/redmambo'
        _isEditor_
                '0'
        _isOwner_
                '1'
        _isSubscriber_
                '0'
        _listAddress_
                'redmambo@dev-sympa.renater.fr'
        _subject_
                'Amical'
1
        _homepage_
                'http://dev-sympa.renater.fr/sympa-dv/info/bluemambo'
        _isEditor_
                '1'
        _isOwner_
                '1'
        _isSubscriber_
                '0'
        _listAddress_
                'bluemambo@dev-sympa.renater.fr'
        _subject_
                'Another mambo. This one is blue.'
2
        _homepage_
                'http://dev-sympa.renater.fr/sympa-dv/info/archeologie'
        _isEditor_
                '1'
        _isOwner_
                '1'
        _isSubscriber_
                '0'
        _listAddress_
                'archeologie@dev-sympa.renater.fr'
        _subject_
                'Liste sur l'archéologie'
3
        _homepage_
                'http://dev-sympa.renater.fr/sympa-dv/info/blackmambo'
        _isEditor_
                '0'
        _isOwner_
                '1'
        _isSubscriber_
                '0'
        _listAddress_
                'blackmambo@dev-sympa.renater.fr'
        _subject_
                'A black mambo'
```

#### amI

Parameters:

  - list name (mandatory): the name of the list for which the function is tested;

  - function (mandatory): the function the existence of which we will test. The allowed values are: `subscriber`, `owner` and `editor`;

  - user (mandatory): the email address of the user for whom we want to know if she has the function indicated in the target list.

Output example:

``` code
param: blackmambo
param: owner
param: david.verdin@renater.fr
Using Session_id 48339436597794


AuthenticateAndRun amI....
0
        '1'
```

#### review

Parameters:

  - the name of the list for which we want the subscribers list (mandatory).

Output example:

``` code
review....
0
        'mail1@renater.fr'
1
        'mail2@renater.fr'
2
        'mail3@renater.fr'
```

#### subscribe

Parameters:

  - list name (mandatory)

Output example:

``` code
subscribe....
0
        '1'
```

#### signoff

Parameters:

  - list name (mandatory)

Output example:

``` code
signoff....
0
        '1'
```

#### add

Parameters:

  - listname (mandatory): the name of the list we want to subscribe the mail address to;

  - email (mandatory): the email to subscribe to the list;

  - gecos: the name under which this email will be subscribed (for example: “John Doe”);

  - quiet: if set to '1', the user doesn't receive a subscription notification

Output example:

``` code
add....
0
        ''
```

#### del

Parameters:

  - listname (mandatory): the name of the list we want to unsubscribe the mail address from;

  - email (mandatory): the email of the user to unsubscribe;

  - quiet: if set to '1', the user doesn't receive an unsubscription notification

Output example:

``` code
del....
0
        '1'
```

#### createList

Parameters:

  - the list name (mandatory);

  - the subject of the list (mandatory);

  - the template to use (mandatory) (the name of a template found in the `create_list_templates` directory for this Sympa robot;

  - the description of the list (mandatory);

  - the topic of the list (mandatory) (one among the different options existing in topics.conf).

Output example:

``` code
param: orangemambo
param: Dude !
param: hotline
param: La liste verte
param: computing
Using Session_id 4860001445687


AuthenticateAndRun createList....
0
        '1'
```

#### closeList

Parameters:

  - the name of the list to close (mandatory).

Output example:

``` code
param: orangemambo
Using Session_id 4860001445687


AuthenticateAndRun closeList....
0
        '1'
```