# NAME

Firefox::Marionette - Automate the Firefox browser with the Marionette protocol

# VERSION

Version 1.23

# SYNOPSIS

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    say $firefox->find_tag('title')->property('innerHTML'); # same as $firefox->title();

    say $firefox->html();

    $firefox->find_class('container-fluid')->find_id('metacpan_search-input')->type('Test::More');

    say "Height of search box is " . $firefox->find_class('container-fluid')->css('height');

    my $file_handle = $firefox->selfie();

    $firefox->find('//button[@name="lucky"]')->click();

    $firefox->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

# DESCRIPTION

This is a client module to automate the Mozilla Firefox browser via the [Marionette protocol](https://developer.mozilla.org/en-US/docs/Mozilla/QA/Marionette/Protocol)

# SUBROUTINES/METHODS

## accept\_alert

accepts a currently displayed modal message box

## accept\_connections

Enables or disables accepting new socket connections.  By calling this method with false the server will not accept any further connections, but existing connections will not be forcible closed. Use true to re-enable accepting connections.

Please note that when closing the connection via the client you can end-up in a non-recoverable state if it hasn't been enabled before.

## active\_element

returns the active element of the current browsing context's document element, if the document element is non-null.

## add\_certificate

accepts a hash as a parameter and adds the specified certificate to the Firefox database with the supplied or default trust.  Allowed keys are below;

- path - a file system path to a single [PEM encoded X.509 certificate](https://datatracker.ietf.org/doc/html/rfc7468#section-5).
- string - a string containg a single [PEM encoded X.509 certificate](https://datatracker.ietf.org/doc/html/rfc7468#section-5)
- trust - This is the [trustargs](https://www.mankier.com/1/certutil#-t) value for [NSS](https://wiki.mozilla.org/NSS).  If defaults to 'C,,';

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

    use Firefox::Marionette();

    my $pem_encoded_string = <<'_PEM_';
    -----BEGIN CERTIFICATE-----
    MII..
    -----END CERTIFICATE-----
    _PEM_
    my $firefox = Firefox::Marionette->new()->add_certificate(string => $pem_encoded_string);

## add\_cookie

accepts a single [cookie](https://metacpan.org/pod/Firefox::Marionette::Cookie) object as the first parameter and adds it to the current cookie jar.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

This method throws an exception if you try to [add a cookie for a different domain than the current document](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Errors/InvalidCookieDomain).

## add\_header

accepts a hash of HTTP headers to include in every future HTTP Request.

    use Firefox::Marionette();
    use UUID();

    my $firefox = Firefox::Marionette->new();
    my $uuid = UUID::uuid();
    $firefox->add_header( 'Track-my-automated-tests' => $uuid );
    $firefox->go('https://metacpan.org/');

these headers are added to any existing headers.  To clear headers, see the [delete\_header](https://metacpan.org/pod/Firefox::Marionette#delete_headers) method

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->delete_header( 'Accept' )->add_header( 'Accept' => 'text/perl' )->go('https://metacpan.org/');

will only send out an [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) header that looks like `Accept: text/perl`.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->add_header( 'Accept' => 'text/perl' )->go('https://metacpan.org/');

by itself, will send out an [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) header that may resemble `Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8, text/perl`. This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## add\_login

accepts a hash of the following keys;

- host - The scheme + hostname of the page where the login applies, for example 'https://www.example.org'.
- user - The username for the login.
- password - The password for the login.
- origin - The scheme + hostname that the form-based login [was submitted to](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-action).  Forms with no [action attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-action) default to submitting to the URL of the page containing the login form, so that is stored here. This field should be omitted (it will be set to undef) for http auth type authentications and "" means to match against any form action.
- realm - The HTTP Realm for which the login was requested. When an HTTP server sends a 401 result, the WWW-Authenticate header includes a realm. See [RFC 2617](https://datatracker.ietf.org/doc/html/rfc2617).  If the realm is not specified, or it was blank, the hostname is used instead. For HTML form logins, this field should not be specified.
- user\_field - The name attribute for the username input in a form. Non-form logins should not specify this field.
- password\_field - The name attribute for the password input in a form. Non-form logins should not specify this field.

or a [Firefox::Marionette::Login](https://metacpan.org/pod/Firefox::Marionette::Login) object as the first parameter and adds the login to the Firefox login database.

    use Firefox::Marionette();
    use UUID();

    my $firefox = Firefox::Marionette->new();

    # for http auth logins

    my $http_auth_login = Firefox::Marionette::Login->new(host => 'https://pause.perl.org', user => 'AUSER', password => 'qwerty', realm => 'PAUSE');
    $firefox->add_login($http_auth_login);
    $firefox->go('https://pause.perl.org/pause/authenquery')->accept_alert(); # this goes to the page and submits the http auth popup

    # for form based login

    $firefox->add_login(host => 'https://github.com', origin => 'https://github.com', user => 'me@example.org', password => 'qwerty', user_field => 'login', password_field => 'password');
    my $form_login = Firefox::Marionette::Login(host => 'https://github.com', user => 'me2@example.org', password => 'uiop[]', user_field => 'login', password_field => 'password');

    # or just directly

    $firefox->add_login(host => 'https://github.com', user => 'me2@example.org', password => 'uiop[]', user_field => 'login', password_field => 'password');

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## add\_site\_header

accepts a host name and a hash of HTTP headers to include in every future HTTP Request that is being sent to that particular host.

    use Firefox::Marionette();
    use UUID();

    my $firefox = Firefox::Marionette->new();
    my $uuid = UUID::uuid();
    $firefox->add_site_header( 'metacpan.org', 'Track-my-automated-tests' => $uuid );
    $firefox->go('https://metacpan.org/');

these headers are added to any existing headers going to the metacpan.org site, but no other site.  To clear site headers, see the [delete\_site\_header](https://metacpan.org/pod/Firefox::Marionette#delete_site_headers) method

## addons

returns if pre-existing addons (extensions/themes) are allowed to run.  This will be true for Firefox versions less than 55, as [-safe-mode](http://kb.mozillazine.org/Command_line_arguments#List_of_command_line_arguments_.28incomplete.29) cannot be automated.

## alert\_text

Returns the message shown in a currently displayed modal message box

## alive

This method returns true or false depending on if the Firefox process is still running.

## application\_type

returns the application type for the Marionette protocol.  Should be 'gecko'.

## async\_script 

accepts a scalar containing a javascript function that is executed in the browser.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

The executing javascript is subject to the [script](https://metacpan.org/pod/Firefox::Marionette::Timeouts#script) timeout, which, by default is 30 seconds.

## attribute 

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and a scalar attribute name as the second parameter.  It returns the initial value of the attribute with the supplied name.  This method will return the initial content, the [property](https://metacpan.org/pod/Firefox::Marionette#property) method will return the current content.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    my $element = $firefox->find_id('search_input');
    !defined $element->attribute('value') or die "attribute is not defined!");
    $element->type('Test::More');
    !defined $element->attribute('value') or die "attribute is still not defined!");

## await

accepts a subroutine reference as a parameter and then executes the subroutine.  If a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception is thrown, this method will sleep for [sleep\_time\_in\_ms](https://metacpan.org/pod/Firefox::Marionette#sleep_time_in_ms) milliseconds and then execute the subroutine again.  When the subroutine executes successfully, it will return what the subroutine returns.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new(sleep_time_in_ms => 5)->go('https://metacpan.org/');

    $firefox->find_id('metacpan_search-input')->type('Test::More');

    $firefox->find_name('lucky')->click();

    $firefox->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

## back

causes the browser to traverse one step backward in the joint history of the current browsing context.  The browser will wait for the one step backward to complete or the session's [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) duration to elapse before returning, which, by default is 5 minutes.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## debug

accept a boolean and return the current value of the debug setting.  This allows the dynamic setting of debug.

## default\_binary\_name

just returns the string 'firefox'.  Only of interest when sub-classing.

## browser\_version

This method returns the current version of firefox.

## bye

accepts a subroutine reference as a parameter and then executes the subroutine.  If the subroutine executes successfully, this method will sleep for [sleep\_time\_in\_ms](https://metacpan.org/pod/Firefox::Marionette#sleep_time_in_ms) milliseconds and then execute the subroutine again.  When a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception is thrown, this method will return [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    $firefox->find_id('metacpan_search-input')->type('Test::More');

    $firefox->find_name('lucky')->click();

    $firefox->bye(sub { $firefox->find_name('lucky') })->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

## capabilities

returns the [capabilities](https://metacpan.org/pod/Firefox::Marionette::Capabilities) of the current firefox binary.  You can retrieve [timeouts](https://metacpan.org/pod/Firefox::Marionette::Timeouts) or a [proxy](https://metacpan.org/pod/Firefox::Marionette::Proxy) with this method.

## certificate\_as\_pem

accepts a [certificate stored in the Firefox database](https://metacpan.org/pod/Firefox::Marionette::Certificate) as a parameter and returns a [PEM encoded X.509 certificate](https://datatracker.ietf.org/doc/html/rfc7468#section-5) as a string.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();

    # Generating a ca-bundle.crt to STDOUT from the current firefox instance

    foreach my $certificate (sort { $a->display_name() cmp $b->display_name } $firefox->certificates()) {
        if ($certificate->is_ca_cert()) {
            print '# ' . $certificate->display_name() . "\n" . $firefox->certificate_as_pem($certificate) . "\n";
        }
    }

The [ca-bundle-for-firefox](https://metacpan.org/pod/ca-bundle-for-firefox) command that is provided as part of this distribution does this.

## certificates

returns a list of all known [certificates in the Firefox database](https://metacpan.org/pod/Firefox::Marionette::Certificate).

    use Firefox::Marionette();
    use v5.10;

    # Sometimes firefox can neglect old certificates.  See https://bugzilla.mozilla.org/show_bug.cgi?id=1710716

    my $firefox = Firefox::Marionette->new();
    foreach my $certificate (grep { $_->is_ca_cert() && $_->not_valid_after() < time } $firefox->certificates()) {
        say "The " . $certificate->display_name() " . certificate has expired and should be removed";
        print 'PEM Encoded Certificate ' . "\n" . $firefox->certificate_as_pem($certificate) . "\n";
    }

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## child\_error

This method returns the $? (CHILD\_ERROR) for the Firefox process, or undefined if the process has not yet exited.

## chrome

changes the scope of subsequent commands to chrome context.  This allows things like interacting with firefox menu's and buttons outside of the browser window.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->chrome();
    $firefox->script(...); # running script in chrome context
    $firefox->content();

See the [context](https://metacpan.org/pod/Firefox::Marionette#context) method for an alternative methods for changing the context.

## chrome\_window\_handle

returns an server-assigned integer identifiers for the current chrome window that uniquely identifies it within this Marionette instance.  This can be used to switch to this window at a later point. This corresponds to a window that may itself contain tabs.  This method is replaced by [window\_handle](https://metacpan.org/pod/Firefox::Marionette#window_handle) and appropriate [context](https://metacpan.org/pod/Firefox::Marionette#context) calls for [Firefox 94 and after](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/94#webdriver_conformance_marionette).

## chrome\_window\_handles

returns identifiers for each open chrome window for tests interested in managing a set of chrome windows and tabs separately.  This method is replaced by [window\_handles](https://metacpan.org/pod/Firefox::Marionette#window_handles) and appropriate [context](https://metacpan.org/pod/Firefox::Marionette#context) calls for [Firefox 94 and after](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/94#webdriver_conformance_marionette).

## clear

accepts a [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and clears any user supplied input

## click

accepts a [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and sends a 'click' to it.  The browser will wait for any page load to complete or the session's [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) duration to elapse before returning, which, by default is 5 minutes.  The [click](https://metacpan.org/pod/Firefox::Marionette#click) method is also used to choose an option in a select dropdown.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new(visible => 1)->go('https://ebay.com');
    my $select = $firefox->find_tag('select');
    foreach my $option ($select->find_tag('option')) {
        if ($option->property('value') == 58058) { # Computers/Tablets & Networking
            $option->click();
        }
    }

## close\_current\_chrome\_window\_handle

closes the current chrome window (that is the entire window, not just the tabs).  It returns a list of still available chrome window handles. You will need to [switch\_to\_window](https://metacpan.org/pod/Firefox::Marionette#switch_to_window) to use another window.

## close\_current\_window\_handle

closes the current window/tab.  It returns a list of still available window/tab handles.

## content

changes the scope of subsequent commands to browsing context.  This is the default for when firefox starts and restricts commands to operating in the browser window only.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->chrome();
    $firefox->script(...); # running script in chrome context
    $firefox->content();

See the [context](https://metacpan.org/pod/Firefox::Marionette#context) method for an alternative methods for changing the context.

## context

accepts a string as the first parameter, which may be either 'content' or 'chrome'.  It returns the context type that is Marionette's current target for browsing context scoped commands.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new();
    if ($firefox->context() eq 'content') {
       say "I knew that was going to happen";
    }
    my $old_context = $firefox->context('chrome');
    $firefox->script(...); # running script in chrome context
    $firefox->context($old_context);

See the [content](https://metacpan.org/pod/Firefox::Marionette#content) and [chrome](https://metacpan.org/pod/Firefox::Marionette#chrome) methods for alternative methods for changing the context.

## cookies

returns the [contents](https://metacpan.org/pod/Firefox::Marionette::Cookie) of the cookie jar in scalar or list context.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://github.com');
    foreach my $cookie ($firefox->cookies()) {
        if (defined $cookie->same_site()) {
            say "Cookie " . $cookie->name() . " has a SameSite of " . $cookie->same_site();
        } else {
            warn "Cookie " . $cookie->name() . " does not have the SameSite attribute defined";
        }
    }

## css

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and a scalar CSS property name as the second parameter.  It returns the value of the computed style for that property.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    say $firefox->find_id('metacpan_search-input')->css('height');

## current\_chrome\_window\_handle 

see [chrome\_window\_handle](https://metacpan.org/pod/Firefox::Marionette#chrome_window_handle).

## delete\_certificate

accepts a [certificate stored in the Firefox database](https://metacpan.org/pod/Firefox::Marionette::Certificate) as a parameter and deletes/distrusts the certificate from the Firefox database.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new();
    foreach my $certificate ($firefox->certificates()) {
        if ($certificate->is_ca_cert()) {
            $firefox->delete_certificate($certificate);
        } else {
            say "This " . $certificate->display_name() " certificate is NOT a certificate authority, therefore it is not being deleted";
        }
    }
    say "Good luck visiting a HTTPS website!";

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## delete\_cookie

deletes a single cookie by name.  Accepts a scalar containing the cookie name as a parameter.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://github.com');
    foreach my $cookie ($firefox->cookies()) {
        warn "Cookie " . $cookie->name() . " is being deleted";
        $firefox->delete_cookie($cookie->name());
    }
    foreach my $cookie ($firefox->cookies()) {
        die "Should be no cookies here now";
    }

## delete\_cookies

here be cookie monsters! This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## delete\_header

accepts a list of HTTP header names to delete from future HTTP Requests.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    $firefox->delete_header( 'User-Agent', 'Accept', 'Accept-Encoding' );

will remove the [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent), [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) and [Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding) headers from all future requests

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## delete\_login

accepts a [login](https://metacpan.org/pod/Firefox::Marionette::Login) as a parameter.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    foreach my $login ($firefox->logins()) {
        if ($login->user() eq 'me@example.org') {
            $firefox->delete_login($login);
        }
    }

will remove the logins with the username matching 'me@example.org'.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## delete\_logins

This method empties the password database.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    $firefox->delete_logins();

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## delete\_session

deletes the current WebDriver session.

## delete\_site\_header

accepts a host name and a list of HTTP headers names to delete from future HTTP Requests.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    $firefox->delete_header( 'metacpan.org', 'User-Agent', 'Accept', 'Accept-Encoding' );

will remove the [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent), [Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) and [Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding) headers from all future requests to metacpan.org.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## developer

returns true if the [current version](https://metacpan.org/pod/Firefox::Marionette#browser_version) of firefox is a [developer edition](https://www.mozilla.org/en-US/firefox/developer/) (does the minor version number end with an 'b\\d+'?) version.

## dismiss\_alert

dismisses a currently displayed modal message box

## download

accepts a filesystem path and returns a matching filehandle.  This is trivial for locally running firefox, but sufficiently complex to justify the method for a remote firefox running over ssh.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new( host => '10.1.2.3' )->go('https://metacpan.org/');

    $firefox->find_class('container-fluid')->find_id('metacpan_search-input')->type('Test::More');

    $firefox->find('//button[@name="lucky"]')->click();

    $firefox->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

    while(!$firefox->downloads()) { sleep 1 }

    foreach my $path ($firefox->downloads()) {

        my $handle = $firefox->download($path);

        # do something with downloaded file handle

    }

## downloading

returns true if any files in [downloads](https://metacpan.org/pod/Firefox::Marionette#downloads) end in `.part`

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    $firefox->find_class('container-fluid')->find_id('metacpan_search-input')->type('Test::More');

    $firefox->find('//button[@name="lucky"]')->click();

    $firefox->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

    while(!$firefox->downloads()) { sleep 1 }

    while($firefox->downloading()) { sleep 1 }

    foreach my $path ($firefox->downloads()) {
        say $path;
    }

## downloads

returns a list of file paths (including partial downloads) of downloads during this Firefox session.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    $firefox->find_class('container-fluid')->find_id('metacpan_search-input')->type('Test::More');

    $firefox->find('//button[@name="lucky"]')->click();

    $firefox->await(sub { $firefox->interactive() && $firefox->find_partial('Download') })->click();

    while(!$firefox->downloads()) { sleep 1 }

    foreach my $path ($firefox->downloads()) {
        say $path;
    }

## error\_message

This method returns a human readable error message describing how the Firefox process exited (assuming it started okay).  On Win32 platforms this information is restricted to exit code.

## execute

This utility method executes a command with arguments and returns STDOUT as a chomped string.  It is a simple method only intended for the Firefox::Marionette::\* modules.

## fill\_login

This method searchs the [Password Manager](https://support.mozilla.org/en-US/kb/password-manager-remember-delete-edit-logins) for an appropriate login for any form on the current page.  The form must match the host, the action attribute and the user and password field names.

    use Firefox::Marionette();
    use IO::Prompt();

    my $firefox = Firefox::Marionette->new();

    my $firefox = Firefox::Marionette->new();

    my $url = 'https://github.com';

    my $user = 'me@example.org';

    my $password = IO::Prompt::prompt(-echo => q[*], "Please enter the password for the $user account when logging into $url:");

    $firefox->add_login(host => $url, user => $user, password => 'qwerty', user_field => 'login', password_field => 'password');

    $firefox->go("$url/login");

    $firefox->fill_login();

## find

accepts an [xpath expression](https://en.wikipedia.org/wiki/XPath) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) that matches this expression.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    $firefox->find('//input[@id="metacpan_search-input"]')->type('Test::More');

    # OR in list context 

    foreach my $element ($firefox->find('//input[@id="metacpan_search-input"]')) {
        $element->type('Test::More');
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has](https://metacpan.org/pod/Firefox::Marionette#has) method.

## find\_id

accepts an [id](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'id' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    $firefox->find_id('metacpan_search-input')->type('Test::More');

    # OR in list context 

    foreach my $element ($firefox->find_id('metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_id](https://metacpan.org/pod/Firefox::Marionette#has_id) method.

## find\_name

This method returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'name' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_name('q')->type('Test::More');

    # OR in list context 

    foreach my $element ($firefox->find_name('q')) {
        $element->type('Test::More');
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_name](https://metacpan.org/pod/Firefox::Marionette#has_name) method.

## find\_class

accepts a [class name](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'class' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_class('form-control home-metacpan_search-input')->type('Test::More');

    # OR in list context 

    foreach my $element ($firefox->find_class('form-control home-metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_class](https://metacpan.org/pod/Firefox::Marionette#has_class) method.

## find\_selector

accepts a [CSS Selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) that matches that selector.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_selector('input.home-metacpan_search-input')->type('Test::More');

    # OR in list context 

    foreach my $element ($firefox->find_selector('input.home-metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_selector](https://metacpan.org/pod/Firefox::Marionette#has_selector) method.

## find\_tag

accepts a [tag name](https://developer.mozilla.org/en-US/docs/Web/API/Element/tagName) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with this tag name.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    my $element = $firefox->find_tag('input');

    # OR in list context 

    foreach my $element ($firefox->find_tag('input')) {
        # do something
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown. For the same functionality that returns undef if no elements are found, see the [has\_tag](https://metacpan.org/pod/Firefox::Marionette#has_tag) method.

## find\_link

accepts a text string as the first parameter and returns the first link [element](https://metacpan.org/pod/Firefox::Marionette::Element) that has a matching link text.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_link('API')->click();

    # OR in list context 

    foreach my $element ($firefox->find_link('API')) {
        $element->click();
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_link](https://metacpan.org/pod/Firefox::Marionette#has_link) method.

## find\_partial

accepts a text string as the first parameter and returns the first link [element](https://metacpan.org/pod/Firefox::Marionette::Element) that has a partially matching link text.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_partial('AP')->click();

    # OR in list context 

    foreach my $element ($firefox->find_partial('AP')) {
        $element->click();
    }

If no elements are found, a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception will be thrown.  For the same functionality that returns undef if no elements are found, see the [has\_partial](https://metacpan.org/pod/Firefox::Marionette#has_partial) method.

## forward

causes the browser to traverse one step forward in the joint history of the current browsing context. The browser will wait for the one step forward to complete or the session's [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) duration to elapse before returning, which, by default is 5 minutes.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## full\_screen

full screens the firefox window. This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## go

Navigates the current browsing context to the given [URI](https://metacpan.org/pod/URI) and waits for the document to load or the session's [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) duration to elapse before returning, which, by default is 5 minutes.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    $firefox->go('https://metacpan.org/'); # will only return when metacpan.org is FULLY loaded (including all images / js / css)

To make the [go](https://metacpan.org/pod/Firefox::Marionette#go) method return quicker, you need to set the [page load strategy](https://metacpan.org/pod/Firefox::Marionette::Capabilities#page_load_strategy) [capability](https://metacpan.org/pod/Firefox::Marionette::Capabilities) to an appropriate value, such as below;

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new( capabilities => Firefox::Marionette::Capabilities->new( page_load_strategy => 'eager' ));
    $firefox->go('https://metacpan.org/'); # will return once the main document has been loaded and parsed, but BEFORE sub-resources (images/stylesheets/frames) have been loaded.

When going directly to a URL that needs to be downloaded, please see [BUGS AND LIMITATIONS](https://metacpan.org/pod/Firefox::Marionette#DOWNLOADING-USING-GO-METHOD) for a necessary workaround.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## har

returns a hashref representing the [http archive](https://en.wikipedia.org/wiki/HAR_\(file_format\)) of the session.  This function is subject to the [script](https://metacpan.org/pod/Firefox::Marionette::Timeouts#script) timeout, which, by default is 30 seconds.  It is also possible for the function to hang (until the [script](https://metacpan.org/pod/Firefox::Marionette::Timeouts#script) timeout) if the original [devtools](https://developer.mozilla.org/en-US/docs/Tools) window is closed.  The hashref has been designed to be accepted by the [Archive::Har](https://metacpan.org/pod/Archive::Har) module.  This function should be considered experimental.  Feedback welcome.

    use Firefox::Marionette();
    use Archive::Har();
    use v5.10;

    my $firefox = Firefox::Marionette->new(visible => 1, debug => 1, har => 1);

    $firefox->go("http://metacpan.org/");

    $firefox->find('//input[@id="metacpan_search-input"]')->type('Test::More');
    $firefox->find_name('lucky')->click();

    my $har = Archive::Har->new();
    $har->hashref($firefox->har());

    foreach my $entry ($har->entries()) {
        say $entry->request()->url() . " spent " . $entry->timings()->connect() . " ms establishing a TCP connection";
    }

## has

accepts an [xpath expression](https://en.wikipedia.org/wiki/XPath) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) that matches this expression.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    if (my $element = $firefox->has('//input[@id="metacpan_search-input"]')) {
        $element->type('Test::More');
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find](https://metacpan.org/pod/Firefox::Marionette#find) method.

## has\_id

accepts an [id](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'id' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    if (my $element = $firefox->has_id('metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_id](https://metacpan.org/pod/Firefox::Marionette#find_id) method.

## has\_name

This method returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'name' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->has_name('q')) {
        $element->type('Test::More');
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_name](https://metacpan.org/pod/Firefox::Marionette#find_name) method.

## has\_class

accepts a [class name](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with a matching 'class' property.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->has_class('form-control home-metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_class](https://metacpan.org/pod/Firefox::Marionette#find_class) method.

## has\_selector

accepts a [CSS Selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) that matches that selector.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->has_selector('input.home-metacpan_search-input')) {
        $element->type('Test::More');
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_selector](https://metacpan.org/pod/Firefox::Marionette#find_selector) method.

## has\_tag

accepts a [tag name](https://developer.mozilla.org/en-US/docs/Web/API/Element/tagName) as the first parameter and returns the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) with this tag name.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->has_tag('input')) {
        # do something
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_tag](https://metacpan.org/pod/Firefox::Marionette#find_tag) method.

## has\_link

accepts a text string as the first parameter and returns the first link [element](https://metacpan.org/pod/Firefox::Marionette::Element) that has a matching link text.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->has_link('API')) {
        $element->click();
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_link](https://metacpan.org/pod/Firefox::Marionette#find_link) method.

## has\_partial

accepts a text string as the first parameter and returns the first link [element](https://metacpan.org/pod/Firefox::Marionette::Element) that has a partially matching link text.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $element = $firefox->find_partial('AP')) {
        $element->click();
    }

If no elements are found, this method will return undef.  For the same functionality that throws a [not found](https://metacpan.org/pod/Firefox::Marionette::Exception::NotFound) exception, see the [find\_partial](https://metacpan.org/pod/Firefox::Marionette#find_partial) method.

## html

returns the page source of the content document.  This page source can be wrapped in html that firefox provides.  See the [json](https://metacpan.org/pod/Firefox::Marionette#json) method for an alternative when dealing with response content types such as application/json and [strip](https://metacpan.org/pod/Firefox::Marionette#strip) for an alterative when dealing with other non-html content types such as text/plain.

    use Firefox::Marionette();
    use v5.10;

    say Firefox::Marionette->new()->go('https://metacpan.org/')->html();

## images

returns a list of all of the following elements;

- [img](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)
- [image inputs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/image)

as [Firefox::Marionette::Image](https://metacpan.org/pod/Firefox::Marionette::Image) objects.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $link = $firefox->images()) {
        say "Found a image with width " . $image->width() . "px and height " . $image->height() . "px from " . $image->URL();
    }

If no elements are found, this method will return undef.

## install

accepts the following as the first parameter;

- path to an [xpi file](https://developer.mozilla.org/en-US/docs/Mozilla/XPI).
- path to a directory containing [firefox extension source code](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Your_first_WebExtension).  This directory will be packaged up as an unsigned xpi file.
- path to a top level file (such as [manifest.json](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension#manifest.json)) in a directory containing [firefox extension source code](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Your_first_WebExtension).  This directory will be packaged up as an unsigned xpi file.

and an optional true/false second parameter to indicate if the xpi file should be a [temporary extension](https://extensionworkshop.com/documentation/develop/temporary-installation-in-firefox/) (just for the existance of this browser instance).  Unsigned xpi files [may only be loaded temporarily](https://wiki.mozilla.org/Add-ons/Extension_Signing) (except for [nightly firefox installations](https://www.mozilla.org/en-US/firefox/channel/desktop/#nightly)).  It returns the GUID for the addon which may be used as a parameter to the [uninstall](https://metacpan.org/pod/Firefox::Marionette#uninstall) method.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();

    my $extension_id = $firefox->install('/full/path/to/gnu_terry_pratchett-0.4-an+fx.xpi');

    # OR downloading and installing source code

    system { 'git' } 'git', 'clone', 'https://github.com/kkapsner/CanvasBlocker.git';

    if ($firefox->nightly()) {

        $extension_id = $firefox->install('./CanvasBlocker'); # permanent install for unsigned packages in nightly firefox

    } else {

        $extension_id = $firefox->install('./CanvasBlocker', 1); # temp install for normal firefox

    }

## interactive

returns true if `document.readyState === "interactive"` or if [loaded](https://metacpan.org/pod/Firefox::Marionette#loaded) is true

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_id('search_input')->type('Type::More');
    $firefox->find('//button[@name="lucky"]')->click();
    while(!$firefox->interactive()) {
        # redirecting to Test::More page
    }

## is\_displayed

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter.  This method returns true or false depending on if the element [is displayed](https://firefox-source-docs.mozilla.org/testing/marionette/internals/interaction.html#interaction.isElementDisplayed).

## is\_enabled

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter.  This method returns true or false depending on if the element [is enabled](https://w3c.github.io/webdriver/#is-element-enabled).

## is\_selected

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter.  This method returns true or false depending on if the element [is selected](https://w3c.github.io/webdriver/#dfn-is-element-selected).  Note that this method only makes sense for [checkbox](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox) or [radio](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio) inputs or [option](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) elements in a [select](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select) dropdown.

## json

returns a [JSON](https://metacpan.org/pod/JSON) object that has been parsed from the page source of the content document.  This is a convenience method that wraps the [strip](https://metacpan.org/pod/Firefox::Marionette#strip) method.

    use Firefox::Marionette();
    use v5.10;

    say Firefox::Marionette->new()->go('https://fastapi.metacpan.org/v1/download_url/Firefox::Marionette")->json()->{version};

## key\_down

accepts a parameter describing a key and returns an action for use in the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that corresponding with that key being depressed.

    use Firefox::Marionette();
    use Firefox::Marionette::Keys qw(:all);

    my $firefox = Firefox::Marionette->new();

    $firefox->chrome()->perform(
                                 $firefox->key_down(CONTROL()),
                                 $firefox->key_down('l'),
                               )->release()->content();

## key\_up

accepts a parameter describing a key and returns an action for use in the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that corresponding with that key being released.

    use Firefox::Marionette();
    use Firefox::Marionette::Keys qw(:all);

    my $firefox = Firefox::Marionette->new();

    $firefox->chrome()->perform(
                                 $firefox->key_down(CONTROL()),
                                 $firefox->key_down('l'),
                                 $firefox->pause(20),
                                 $firefox->key_up('l'),
                                 $firefox->key_up(CONTROL())
                               )->content();

## loaded

returns true if `document.readyState === "complete"`

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    $firefox->find_id('search_input')->type('Type::More');
    $firefox->find('//button[@name="lucky"]')->click();
    while(!$firefox->loaded()) {
        # redirecting to Test::More page
    }

## logins\_from\_csv

accepts a filehandle as a parameter and then reads the filehandle for exported logins as CSV.  This is known to work with the following formats;

- [Bitwarden CSV](https://bitwarden.com/help/article/condition-bitwarden-import/)
- [LastPass CSV](https://support.logmeininc.com/lastpass/help/how-do-i-nbsp-export-stored-data-from-lastpass-using-a-generic-csv-file)
- [KeePass CSV](https://keepass.info/help/base/importexport.html#csv)

returns a list of [Firefox::Marionette::Login](https://metacpan.org/pod/Firefox::Marionette::Login) objects.

    use Firefox::Marionette();
    use FileHandle();

    my $handle = FileHandle->new('/path/to/last_pass.csv');
    my $firefox = Firefox::Marionette->new();
    foreach my $login (Firefox::Marionette->logins_from_csv($handle)) {
        $firefox->add_login($login);
    }

## logins\_from\_xml

accepts a filehandle as a parameter and then reads the filehandle for exported logins as XML.  This is known to work with the following formats;

- [KeePass 1.x XML](https://keepass.info/help/base/importexport.html#xml)

returns a list of [Firefox::Marionette::Login](https://metacpan.org/pod/Firefox::Marionette::Login) objects.

    use Firefox::Marionette();
    use FileHandle();

    my $handle = FileHandle->new('/path/to/keepass1.xml');
    my $firefox = Firefox::Marionette->new();
    foreach my $login (Firefox::Marionette->logins_from_csv($handle)) {
        $firefox->add_login($login);
    }

## logins\_from\_zip

accepts a filehandle as a parameter and then reads the filehandle for exported logins as a zip file.  This is known to work with the following formats;

- [1Password Unencrypted Export format](https://support.1password.com/1pux-format/)

returns a list of [Firefox::Marionette::Login](https://metacpan.org/pod/Firefox::Marionette::Login) objects.

    use Firefox::Marionette();
    use FileHandle();

    my $handle = FileHandle->new('/path/to/1Passwordv8.1pux');
    my $firefox = Firefox::Marionette->new();
    foreach my $login (Firefox::Marionette->logins_from_zip($handle)) {
        $firefox->add_login($login);
    }

## links

returns a list of all of the following elements;

- [anchor](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)
- [area](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/area)
- [frame](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/frame)
- [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)
- [meta](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta)

as [Firefox::Marionette::Link](https://metacpan.org/pod/Firefox::Marionette::Link) objects.

This method is subject to the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeouts#implicit) timeout, which, by default is 0 seconds.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    if (my $link = $firefox->links()) {
        if ($link->tag() eq 'a') {
            warn "Found a hyperlink to " . $link->URL();
        }
    }

If no elements are found, this method will return undef.

## macos\_binary\_paths

returns a list of filesystem paths that this module will check for binaries that it can automate when running on [MacOS](https://en.wikipedia.org/wiki/MacOS).  Only of interest when sub-classing.

## marionette\_protocol

returns the version for the Marionette protocol.  Current most recent version is '3'.

## maximise

maximises the firefox window. This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## mime\_types

returns a list of MIME types that will be downloaded by firefox and made available from the [downloads](https://metacpan.org/pod/Firefox::Marionette#downloads) method

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new(mime_types => [ 'application/pkcs10' ])

    foreach my $mime_type ($firefox->mime_types()) {
        say $mime_type;
    }

## minimise

minimises the firefox window. This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## mouse\_down

accepts a parameter describing which mouse button the method should apply to ([left](https://metacpan.org/pod/Firefox::Marionette::Buttons#LEFT), [middle](https://metacpan.org/pod/Firefox::Marionette::Buttons#MIDDLE) or [right](https://metacpan.org/pod/Firefox::Marionette::Buttons#RIGHT)) and returns an action for use in the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that corresponding with a mouse button being depressed.

## mouse\_move

accepts a [element](https://metacpan.org/pod/Firefox::Marionette::Element) parameter, or a `( x => 0, y => 0 )` type hash manually describing exactly where to move the mouse to and returns an action for use in the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that corresponding with such a mouse movement, either to the specified co-ordinates or to the middle of the supplied [element](https://metacpan.org/pod/Firefox::Marionette::Element) parameter.  Other parameters that may be passed are listed below;

- origin - the origin of the C(&lt;x => 0, y => 0)> co-ordinates.  Should be either `viewport`, `pointer` or an [element](https://metacpan.org/pod/Firefox::Marionette::Element).
- duration - Number of milliseconds over which to distribute the move. If not defined, the duration defaults to 0.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## mouse\_up

accepts a parameter describing which mouse button the method should apply to ([left](https://metacpan.org/pod/Firefox::Marionette::Buttons#LEFT), [middle](https://metacpan.org/pod/Firefox::Marionette::Buttons#MIDDLE) or [right](https://metacpan.org/pod/Firefox::Marionette::Buttons#RIGHT)) and returns an action for use in the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that corresponding with a mouse button being released.

## new

accepts an optional hash as a parameter.  Allowed keys are below;

- addons - should any firefox extensions and themes be available in this session.  This defaults to "0".
- binary - use the specified path to the [Firefox](https://firefox.org/) binary, rather than the default path.
- capabilities - use the supplied [capabilities](https://metacpan.org/pod/Firefox::Marionette::Capabilities) object, for example to set whether the browser should [accept insecure certs](https://metacpan.org/pod/Firefox::Marionette::Capabilities#accept_insecure_certs) or whether the browser should use a [proxy](https://metacpan.org/pod/Firefox::Marionette::Proxy).
- chatty - Firefox is extremely chatty on the network, including checking for the lastest malware/phishing sites, updates to firefox/etc.  This option is therefore off ("0") by default, however, it can be switched on ("1") if required.  Even with chatty switched off, [connections to firefox.settings.services.mozilla.com will still be made](https://bugzilla.mozilla.org/show_bug.cgi?id=1598562#c13).  The only way to prevent this seems to be to set firefox.settings.services.mozilla.com to 127.0.0.1 via [/etc/hosts](https://en.wikipedia.org/wiki//etc/hosts).  NOTE: that this option only works when profile\_name/profile is not specified.
- console - show the [browser console](https://developer.mozilla.org/en-US/docs/Tools/Browser_Console/) when the browser is launched.  This defaults to "0" (off).
- debug - should firefox's debug to be available via STDERR. This defaults to "0". Any ssh connections will also be printed to STDERR.  This defaults to "0" (off).  This setting may be updated by the [debug](https://metacpan.org/pod/Firefox::Marionette#debug) method.
- developer - only allow a [developer edition](https://www.mozilla.org/en-US/firefox/developer/) to be launched. This defaults to "0" (off).
- devtools - begin the session with the [devtools](https://developer.mozilla.org/en-US/docs/Tools) window opened in a separate window.
- height - set the [height](http://kb.mozillazine.org/Command_line_arguments#List_of_command_line_arguments_.28incomplete.29) of the initial firefox window
- har - begin the session with the [devtools](https://developer.mozilla.org/en-US/docs/Tools) window opened in a separate window.  The [HAR Export Trigger](https://addons.mozilla.org/en-US/firefox/addon/har-export-trigger/) addon will be loaded into the new session automatically, which means that [-safe-mode](http://kb.mozillazine.org/Command_line_arguments#List_of_command_line_arguments_.28incomplete.29) will not be activated for this session AND this functionality will only be available for Firefox 61+.
- host - use [ssh](https://man.openbsd.org/ssh.1) to create and automate firefox on the specified host.  See [REMOTE AUTOMATION OF FIREFOX VIA SSH](https://metacpan.org/pod/Firefox::Marionette#REMOTE-AUTOMATION-OF-FIREFOX-VIA-SSH).
- implicit - a shortcut to allow directly providing the [implicit](https://metacpan.org/pod/Firefox::Marionette::Timeout#implicit) timeout, instead of needing to use timeouts from the capabilities parameter.  Overrides all longer ways.
- kiosk - start the browser in [kiosk](https://support.mozilla.org/en-US/kb/firefox-enterprise-kiosk-mode) mode.
- mime\_types - any MIME types that Firefox will encounter during this session.  MIME types that are not specified will result in a hung browser (the File Download popup will appear).
- nightly - only allow a [nightly release](https://www.mozilla.org/en-US/firefox/channel/desktop/#nightly) to be launched.  This defaults to "0" (off).
- port - if the "host" parameter is also set, use [ssh](https://man.openbsd.org/ssh.1) to create and automate firefox via the specified port.  See [REMOTE AUTOMATION OF FIREFOX VIA SSH](https://metacpan.org/pod/Firefox::Marionette#REMOTE-AUTOMATION-OF-FIREFOX-VIA-SSH).
- page\_load - a shortcut to allow directly providing the [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) timeout, instead of needing to use timeouts from the capabilities parameter.  Overrides all longer ways.
- profile - create a new profile based on the supplied [profile](https://metacpan.org/pod/Firefox::Marionette::Profile).  NOTE: firefox ignores any changes made to the profile on the disk while it is running.
- profile\_name - pick a specific existing profile to automate, rather than creating a new profile.  [Firefox](https://firefox.com) refuses to allow more than one instance of a profile to run at the same time.  Profile names can be obtained by using the [Firefox::Marionette::Profile::names()](https://metacpan.org/pod/Firefox::Marionette::Profile#names) method.  NOTE: firefox ignores any changes made to the profile on the disk while it is running.
- reconnect - an experimental parameter to allow a reconnection to firefox that a connection has been discontinued.  See the survive parameter.
- script - a shortcut to allow directly providing the [script](https://metacpan.org/pod/Firefox::Marionette::Timeout#script) timeout, instead of needing to use timeouts from the capabilities parameter.  Overrides all longer ways.
- seer - this option is switched off "0" by default.  When it is switched on "1", it will activate the various speculative and pre-fetch options for firefox.  NOTE: that this option only works when profile\_name/profile is not specified.
- sleep\_time\_in\_ms - the amount of time (in milliseconds) that this module should sleep when unsuccessfully calling the subroutine provided to the [await](https://metacpan.org/pod/Firefox::Marionette#await) or [bye](https://metacpan.org/pod/Firefox::Marionette#bye) methods.  This defaults to "1" millisecond.
- survive - if this is set to a true value, firefox will not automatically exit when the object goes out of scope.  See the reconnect parameter for an experimental technique for reconnecting.
- trust - give a path to a [root certificate](https://en.wikipedia.org/wiki/Root_certificate) encoded as a [PEM encoded X.509 certificate](https://datatracker.ietf.org/doc/html/rfc7468#section-5) that will be trusted for this session.
- timeouts - a shortcut to allow directly providing a [timeout](https://metacpan.org/pod/Firefox::Marionette::Timeout) object, instead of needing to use timeouts from the capabilities parameter.  Overrides the timeouts provided (if any) in the capabilities parameter.
- user - if the "host" parameter is also set, use [ssh](https://man.openbsd.org/ssh.1) to create and automate firefox with the specified user.  See [REMOTE AUTOMATION OF FIREFOX VIA SSH](https://metacpan.org/pod/Firefox::Marionette#REMOTE-AUTOMATION-OF-FIREFOX-VIA-SSH).  The user will default to the current user name.
- visible - should firefox be visible on the desktop.  This defaults to "0".
- waterfox - only allow a binary that looks like a [waterfox version](https://www.waterfox.net/) to be launched.
- width - set the [width](http://kb.mozillazine.org/Command_line_arguments#List_of_command_line_arguments_.28incomplete.29) of the initial firefox window

This method returns a new `Firefox::Marionette` object, connected to an instance of [firefox](https://firefox.com).  In a non MacOS/Win32/Cygwin environment, if necessary (no DISPLAY variable can be found and the visible parameter to the new method has been set to true) and possible (Xvfb can be executed successfully), this method will also automatically start an [Xvfb](https://en.wikipedia.org/wiki/Xvfb) instance.

    use Firefox::Marionette();

    my $remote_darwin_firefox = Firefox::Marionette->new(
                     debug => 1,
                     host => '10.1.2.3',
                     trust => '/path/to/root_ca.pem',
                     binary => '/Applications/Firefox.app/Contents/MacOS/firefox'
                                                        ); # start a temporary profile for a remote firefox and load a new CA into the temp profile
    ...

    foreach my $profile_name (Firefox::Marionette::Profile->names()) {
        my $firefox_with_existing_profile = Firefox::Marionette->new( profile_name => $profile_name, visible => 1 );
        ...
    }

## new\_window

accepts an optional hash as the parameter.  Allowed keys are below;

- focus - a boolean field representing if the new window be opened in the foreground (focused) or background (not focused). Defaults to false.
- private - a boolean field representing if the new window should be a private window. Defaults to false.
- type - the type of the new window. Can be one of 'tab' or 'window'. Defaults to 'tab'.

Returns the window handle for the new window.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();

    my $window_handle = $firefox->new_window(type => 'tab');

    $firefox->switch_to_window($window_handle);

## new\_session

creates a new WebDriver session.  It is expected that the caller performs the necessary checks on the requested capabilities to be WebDriver conforming.  The WebDriver service offered by Marionette does not match or negotiate capabilities beyond type and bounds checks.

## nightly

returns true if the [current version](https://metacpan.org/pod/Firefox::Marionette#browser_version) of firefox is a [nightly release](https://www.mozilla.org/en-US/firefox/channel/desktop/#nightly) (does the minor version number end with an 'a1'?)

## paper\_sizes 

returns a list of all the recognised names for paper sizes, such as A4 or LEGAL.

## pause

accepts a parameter in milliseconds and returns a corresponding action for the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method that will cause a pause in the chain of actions given to the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method.

## pdf

accepts a optional hash as the first parameter with the following allowed keys;

- landscape - Paper orientation.  Boolean value.  Defaults to false
- margin - A hash describing the margins.  The hash may have the following optional keys, 'top', 'left', 'right' and 'bottom'.  All these keys are in cm and default to 1 (~0.4 inches)
- page - A hash describing the page.  The hash may have the following keys; 'height' and 'width'.  Both keys are in cm and default to US letter size.  See the 'size' key.
- page\_ranges - A list of the pages to print. Available for [Firefox 96](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/96#webdriver_conformance_marionette) and after.
- print\_background - Print background graphics.  Boolean value.  Defaults to false. 
- raw - rather than a file handle containing the PDF, the binary PDF will be returned.
- scale - Scale of the webpage rendering.  Defaults to 1.
- size - The desired size (width and height) of the pdf, specified by name.  See the page key for an alternative and the [paper\_sizes](https://metacpan.org/pod/Firefox::Marionette#paper_sizes) method for a list of accepted page size names. 
- shrink\_to\_fit - Whether or not to override page size as defined by CSS.  Boolean value.  Defaults to true. 

returns a [File::Temp](https://metacpan.org/pod/File::Temp) object containing a PDF encoded version of the current page for printing.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    my $handle = $firefox->pdf();
    foreach my $paper_size ($firefox->paper_sizes()) {
            $handle = $firefox->pdf(size => $paper_size, landscape => 1, margin => { top => 0.5, left => 1.5 });
            ...
            print $firefox->pdf(page => { width => 21, height => 27 }, raw => 1);
            ...
    }

## perform

accepts a list of actions (see [mouse\_up](https://metacpan.org/pod/Firefox::Marionette#mouse_up), [mouse\_down](https://metacpan.org/pod/Firefox::Marionette#mouse_down), [mouse\_move](https://metacpan.org/pod/Firefox::Marionette#mouse_move), [pause](https://metacpan.org/pod/Firefox::Marionette#pause), [key\_down](https://metacpan.org/pod/Firefox::Marionette#key_down) and [key\_up](https://metacpan.org/pod/Firefox::Marionette#key_up)) and performs these actions in sequence.  This allows fine control over interactions, including sending right clicks to the browser and sending Control, Alt and other special keys.  The [release](https://metacpan.org/pod/Firefox::Marionette#release) method will complete outstanding actions (such as [mouse\_up](https://metacpan.org/pod/Firefox::Marionette#mouse_up) or [key\_up](https://metacpan.org/pod/Firefox::Marionette#key_up) actions).

    use Firefox::Marionette();
    use Firefox::Marionette::Keys qw(:all);
    use Firefox::Marionette::Buttons qw(:all);

    my $firefox = Firefox::Marionette->new();

    $firefox->chrome()->perform(
                                 $firefox->key_down(CONTROL()),
                                 $firefox->key_down('l'),
                                 $firefox->key_up('l'),
                                 $firefox->key_up(CONTROL())
                               )->content();

    $firefox->go('https://metacpan.org');
    my $help_button = $firefox->find_class('btn search-btn help-btn');
    $firefox->perform(
                                  $firefox->mouse_move($help_button),
                                  $firefox->mouse_down(RIGHT_BUTTON()),
                                  $firefox->pause(4),
                                  $firefox->mouse_up(RIGHT_BUTTON()),
                );

See the [release](https://metacpan.org/pod/Firefox::Marionette#release) method for an alternative for manually specifying all the [mouse\_up](https://metacpan.org/pod/Firefox::Marionette#mouse_up) and [key\_up](https://metacpan.org/pod/Firefox::Marionette#key_up) methods

## profile\_directory

returns the profile directory used by the current instance of firefox.  This is mainly intended for debugging firefox.  Firefox is not designed to cope with these files being altered while firefox is running.

## property

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and a scalar attribute name as the second parameter.  It returns the current value of the property with the supplied name.  This method will return the current content, the [attribute](https://metacpan.org/pod/Firefox::Marionette#attribute) method will return the initial content.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');
    my $element = $firefox->find_id('search_input');
    $element->property('value') eq '' or die "Initial property is the empty string";
    $element->type('Test::More');
    $element->property('value') eq 'Test::More' or die "This property should have changed!";

    # OR getting the innerHTML property

    my $title = $firefox->find_tag('title')->property('innerHTML'); # same as $firefox->title();

## pwd\_mgr\_lock

Accepts a new [primary password](https://support.mozilla.org/en-US/kb/use-primary-password-protect-stored-logins) and locks the [Password Manager](https://support.mozilla.org/en-US/kb/password-manager-remember-delete-edit-logins) with it.

    use Firefox::Marionette();
    use IO::Prompt();

    my $firefox = Firefox::Marionette->new();
    my $password = IO::Prompt::prompt(-echo => q[*], "Please enter the password for the Firefox Password Manager:");
    $firefox->pwd_mgr_lock($password);
    $firefox->pwd_mgr_logout();
    # now no-one can access the Password Manager Database without the value in $password

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## pwd\_mgr\_login

Accepts the [primary password](https://support.mozilla.org/en-US/kb/use-primary-password-protect-stored-logins) and allows the user to access the [Password Manager](https://support.mozilla.org/en-US/kb/password-manager-remember-delete-edit-logins).

    use Firefox::Marionette();
    use IO::Prompt();

    my $firefox = Firefox::Marionette->new( profile_name => 'default' );
    my $password = IO::Prompt::prompt(-echo => q[*], "Please enter the password for the Firefox Password Manager:");
    $firefox->pwd_mgr_login($password);
    ...
    # access the Password Database.
    ...
    $firefox->pwd_mgr_logout();
    ...
    # no longer able to access the Password Database.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## pwd\_mgr\_logout

Logs the user out of being able to access the [Password Manager](https://support.mozilla.org/en-US/kb/password-manager-remember-delete-edit-logins).

    use Firefox::Marionette();
    use IO::Prompt();

    my $firefox = Firefox::Marionette->new( profile_name => 'default' );
    my $password = IO::Prompt::prompt(-echo => q[*], "Please enter the password for the Firefox Password Manager:");
    $firefox->pwd_mgr_login($password);
    ...
    # access the Password Database.
    ...
    $firefox->pwd_mgr_logout();
    ...
    # no longer able to access the Password Database.

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## pwd\_mgr\_needs\_login

returns true or false if the [Password Manager](https://support.mozilla.org/en-US/kb/password-manager-remember-delete-edit-logins) has been locked and needs a [primary password](https://support.mozilla.org/en-US/kb/use-primary-password-protect-stored-logins) to access it.

    use Firefox::Marionette();
    use IO::Prompt();

    my $firefox = Firefox::Marionette->new( profile_name => 'default' );
    if ($firefox->pwd_mgr_needs_login()) {
      my $password = IO::Prompt::prompt(-echo => q[*], "Please enter the password for the Firefox Password Manager:");
      $firefox->pwd_mgr_login($password);
    }

## quit

Marionette will stop accepting new connections before ending the current session, and finally attempting to quit the application.  This method returns the $? (CHILD\_ERROR) value for the Firefox process

## rect

accepts a [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and returns the current [position and size](https://metacpan.org/pod/Firefox::Marionette::Element::Rect) of the [element](https://metacpan.org/pod/Firefox::Marionette::Element)

## refresh

refreshes the current page.  The browser will wait for the page to completely refresh or the session's [page\_load](https://metacpan.org/pod/Firefox::Marionette::Timeouts#page_load) duration to elapse before returning, which, by default is 5 minutes.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## release

completes any outstanding actions issued by the [perform](https://metacpan.org/pod/Firefox::Marionette#perform) method.

    use Firefox::Marionette();
    use Firefox::Marionette::Keys qw(:all);
    use Firefox::Marionette::Buttons qw(:all);

    my $firefox = Firefox::Marionette->new();

    $firefox->chrome()->perform(
                                 $firefox->key_down(CONTROL()),
                                 $firefox->key_down('l'),
                               )->release()->content();

    $firefox->go('https://metacpan.org');
    my $help_button = $firefox->find_class('btn search-btn help-btn');
    $firefox->perform(
                                  $firefox->mouse_move($help_button),
                                  $firefox->mouse_down(RIGHT_BUTTON()),
                                  $firefox->pause(4),
                )->release();

## restart

restarts the browser.  After the restart, [capabilities](https://metacpan.org/pod/Firefox::Marionette::Capabilities) should be restored.  The same profile settings should be applied, but the current state of the browser (such as the [uri](https://metacpan.org/pod/Firefox::Marionette#uri) will be reset (like after a normal browser restart).  This method is primarily intended for use by the [update](https://metacpan.org/pod/Firefox::Marionette#update) method.  Not sure if this is useful by itself.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();

    $firefox->restart(); # but why?

This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## root\_directory

this is the root directory for the current instance of firefox.  The directory may exist on a remote server.  For debugging purposes only.

## screen\_orientation

returns the current browser orientation.  This will be one of the valid primary orientation values 'portrait-primary', 'landscape-primary', 'portrait-secondary', or 'landscape-secondary'.  This method is only currently available on Android (Fennec).

## script 

accepts a scalar containing a javascript function body that is executed in the browser, and an optional hash as a second parameter.  Allowed keys are below;

- args - The reference to a list is the arguments passed to the function body.
- filename - Filename of the client's program where this script is evaluated.
- line - Line in the client's program where this script is evaluated.
- new - Forces the script to be evaluated in a fresh sandbox.  Note that if it is undefined, the script will normally be evaluted in a fresh sandbox.
- sandbox - Name of the sandbox to evaluate the script in.  The sandbox is cached for later re-use on the same [window](https://developer.mozilla.org/en-US/docs/Web/API/Window) object if `new` is false.  If he parameter is undefined, the script is evaluated in a mutable sandbox.  If the parameter is "system", it will be evaluted in a sandbox with elevated system privileges, equivalent to chrome space.
- timeout - A timeout to override the default [script](https://metacpan.org/pod/Firefox::Marionette::Timeouts#script) timeout, which, by default is 30 seconds.

Returns the result of the javascript function.  When a parameter is an [element](https://metacpan.org/pod/Firefox::Marionette::Element) (such as being returned from a [find](https://metacpan.org/pod/Firefox::Marionette#find) type operation), the [script](https://metacpan.org/pod/Firefox::Marionette#script) method will automatically translate that into a javascript object.  Likewise, when the result being returned in a [script](https://metacpan.org/pod/Firefox::Marionette#script) method is an [element](https://dom.spec.whatwg.org/#concept-element) it will be automatically translated into a [perl object](https://metacpan.org/pod/Firefox::Marionette::Element).

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new()->go('https://metacpan.org/');

    if (my $element = $firefox->script('return document.getElementsByName("lucky")[0];')) {
        say "Lucky find is a " . $element->tag_name() . " element";
    }

    my $search_input = $firefox->find_id('metacpan_search-input');

    $firefox->script('arguments[0].style.backgroundColor = "red"', args => [ $search_input ]); # turn the search input box red

The executing javascript is subject to the [script](https://metacpan.org/pod/Firefox::Marionette::Timeouts#script) timeout, which, by default is 30 seconds.

## selfie

returns a [File::Temp](https://metacpan.org/pod/File::Temp) object containing a lossless PNG image screenshot.  If an [element](https://metacpan.org/pod/Firefox::Marionette::Element) is passed as a parameter, the screenshot will be restricted to the element.  

If an [element](https://metacpan.org/pod/Firefox::Marionette::Element) is not passed as a parameter and the current [context](https://metacpan.org/pod/Firefox::Marionette#context) is 'chrome', a screenshot of the current viewport will be returned.

If an [element](https://metacpan.org/pod/Firefox::Marionette::Element) is not passed as a parameter and the current [context](https://metacpan.org/pod/Firefox::Marionette#context) is 'content', a screenshot of the current frame will be returned.

The parameters after the [element](https://metacpan.org/pod/Firefox::Marionette::Element) parameter are taken to be a optional hash with the following allowed keys;

- hash - return a SHA256 hex encoded digest of the PNG image rather than the image itself
- full - take a screenshot of the whole document unless the first [element](https://metacpan.org/pod/Firefox::Marionette::Element) parameter has been supplied.
- raw - rather than a file handle containing the screenshot, the binary PNG image will be returned.
- scroll - scroll to the [element](https://metacpan.org/pod/Firefox::Marionette::Element) supplied
- highlights - a reference to a list containing [elements](https://metacpan.org/pod/Firefox::Marionette::Element) to draw a highlight around.  Not available in [Firefox 70](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/70#WebDriver_conformance_Marionette) onwards.

## send\_alert\_text

sends keys to the input field of a currently displayed modal message box

## shadow\_root

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as a parameter and returns it's [ShadowRoot](https://developer.mozilla.org/en-US/docs/Web/API/ShadowRoot) as a [shadow root](https://metacpan.org/pod/Firefox::Marionette::ShadowRoot) object or throws an exception.

    use Firefox::Marionette();
    use Cwd();

    my $firefox = Firefox::Marionette->new()->go('file://' . Cwd::cwd() . '/t/data/elements.html');

    $firefox->find_class('add')->click();
    my $custom_square = $firefox->find_tag('custom-square');
    my $shadow_root = $firefox->shadow_root($custom_square);

    foreach my $element (@{$firefox->script('return arguments[0].children', args => [ $shadow_root ])}) {
        warn $element->tag_name();
    }

## shadowy

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as a parameter and returns true if the element has a [ShadowRoot](https://developer.mozilla.org/en-US/docs/Web/API/ShadowRoot) or false otherwise.

    use Firefox::Marionette();
    use Cwd();

    my $firefox = Firefox::Marionette->new()->go('file://' . Cwd::cwd() . '/t/data/elements.html');
    $firefox->find_class('add')->click();
    my $custom_square = $firefox->find_tag('custom-square');
    if ($firefox->shadowy($custom_square)) {
        my $shadow_root = $firefox->find_tag('custom-square')->shadow_root();
        warn $firefox->script('return arguments[0].innerHTML', args => [ $shadow_root ]);
        ...
    }

This function will probably be used to see if the [shadow\_root](https://metacpan.org/pod/Firefox::Marionette::Element#shadow_root) method can be called on this element without raising an exception.

## sleep\_time\_in\_ms

accepts a new time to sleep in [await](https://metacpan.org/pod/Firefox::Marionette#await) or [bye](https://metacpan.org/pod/Firefox::Marionette#bye) methods and returns the previous time.  The default time is "1" millisecond.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new(sleep_time_in_ms => 5); # setting default time to 5 milliseconds

    my $old_time_in_ms = $firefox->sleep_time_in_ms(8); # setting default time to 8 milliseconds, returning 5 (milliseconds)

## ssh\_local\_directory

returns the path to the local directory for the ssh connection (if any). For debugging purposes only.

## strip

returns the page source of the content document after an attempt has been made to remove typical firefox html wrappers of non html content types such as text/plain and application/json.  See the [json](https://metacpan.org/pod/Firefox::Marionette#json) method for an alternative when dealing with response content types such as application/json and [html](https://metacpan.org/pod/Firefox::Marionette#html) for an alterative when dealing with html content types.  This is a convenience method that wraps the [html](https://metacpan.org/pod/Firefox::Marionette#html) method.

    use Firefox::Marionette();
    use JSON();
    use v5.10;

    say JSON::decode_json(Firefox::Marionette->new()->go("https://fastapi.metacpan.org/v1/download_url/Firefox::Marionette")->strip())->{version};

Note that this method will assume the bytes it receives from the [html](https://metacpan.org/pod/Firefox::Marionette#html) method are UTF-8 encoded and will translate accordingly, throwing an exception in the process if the bytes are not UTF-8 encoded.

## switch\_to\_frame

accepts a [frame](https://metacpan.org/pod/Firefox::Marionette::Element) as a parameter and switches to it within the current window.

## switch\_to\_parent\_frame

set the current browsing context for future commands to the parent of the current browsing context

## switch\_to\_window

accepts a window handle (either the result of [window\_handles](https://metacpan.org/pod/Firefox::Marionette#window_handles) or a window name as a parameter and switches focus to this window.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();
    $firefox->version
    my $original_window_uuid = $firefox->window_handle();
    $firefox->new_window( type => 'tab' );
    $firefox->new_window( type => 'window' );
    $firefox->switch_to_window($original_window_uuid);
    $firefox->go('https://metacpan.org');

## tag\_name

accepts a [Firefox::Marionette::Element](https://metacpan.org/pod/Firefox::Marionette::Element) object as the first parameter and returns the relevant tag name.  For example '[a](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)' or '[input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)'.

## text

accepts a [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and returns the text that is contained by that element (if any)

## timeouts

returns the current [timeouts](https://metacpan.org/pod/Firefox::Marionette::Timeouts) for page loading, searching, and scripts.

## title

returns the current [title](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title) of the window.

## type

accepts an [element](https://metacpan.org/pod/Firefox::Marionette::Element) as the first parameter and a string as the second parameter.  It sends the string to the specified [element](https://metacpan.org/pod/Firefox::Marionette::Element) in the current page, such as filling out a text box. This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

## update

queries the Update Services and applies any available updates.  [Restarts](https://metacpan.org/pod/Firefox::Marionette#restart) the browser if necessary to complete the update.  This function is experimental and currently has not been successfully tested on Win32 or MacOS.

    use Firefox::Marionette();
    use v5.10;

    my $firefox = Firefox::Marionette->new();

    my $update = $firefox->update();

    while($update->successful()) {
        $update = $firefox->update();
    }

    say "Updated to " . $update->display_version() . " - Build ID " . $update->build_id();

    $firefox->quit();

returns a [status](https://metacpan.org/pod/Firefox::Marionette::UpdateStatus) object that contains useful information about any updates that occurred.

## uninstall

accepts the GUID for the addon to uninstall.  The GUID is returned when from the [install](https://metacpan.org/pod/Firefox::Marionette#install) method.  This method returns [itself](https://metacpan.org/pod/Firefox::Marionette) to aid in chaining methods.

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new();

    my $extension_id = $firefox->install('/full/path/to/gnu_terry_pratchett-0.4-an+fx.xpi');

    # do something

    $firefox->uninstall($extension_id); # not recommended to uninstall this extension IRL.

## uri

returns the current [URI](https://metacpan.org/pod/URI) of current top level browsing context for Desktop.  It is equivalent to the javascript `document.location.href`

## win32\_organisation

accepts a parameter of a Win32 product name and returns the matching organisation.  Only of interest when sub-classing.

## win32\_product\_names

returns a hash of known Windows product names (such as 'Mozilla Firefox') with priority orders.  The lower the priority will determine the order that this module will check for the existance of this product.  Only of interest when sub-classing.

## window\_handle

returns the current window's handle. On desktop this typically corresponds to the currently selected tab.  returns an opaque server-assigned identifier to this window that uniquely identifies it within this Marionette instance.  This can be used to switch to this window at a later point.

    use Firefox::Marionette();
    use 5.010;

    my $firefox = Firefox::Marionette->new();
    my $original_window_uuid = $firefox->window_handle();

## window\_handles

returns a list of top-level browsing contexts. On desktop this typically corresponds to the set of open tabs for browser windows, or the window itself for non-browser chrome windows.  Each window handle is assigned by the server and is guaranteed unique, however the return array does not have a specified ordering.

    use Firefox::Marionette();
    use 5.010;

    my $firefox = Firefox::Marionette->new();
    my $original_window_uuid = $firefox->window_handle();
    $firefox->new_window( type => 'tab' );
    $firefox->new_window( type => 'window' );
    say "There are " . $firefox->window_handles() . " tabs open in total";
    say "Across " . $firefox->chrome()->window_handles()->content() . " chrome windows";

## window\_rect

accepts an optional [position and size](https://metacpan.org/pod/Firefox::Marionette::Window::Rect) as a parameter, sets the current browser window to that position and size and returns the previous [position, size and state](https://metacpan.org/pod/Firefox::Marionette::Window::Rect) of the browser window.  If no parameter is supplied, it returns the current  [position, size and state](https://metacpan.org/pod/Firefox::Marionette::Window::Rect) of the browser window.

## window\_type

returns the current window's type.  This should be 'navigator:browser'.

## xvfb\_pid

returns the pid of the xvfb process if it exists.

## xvfb\_display

returns the value for the DISPLAY environment variable if one has been generated for the xvfb environment.

## xvfb\_xauthority

returns the value for the XAUTHORITY environment variable if one has been generated for the xvfb environment

# AUTOMATING THE FIREFOX PASSWORD MANAGER

This module allows you to login to a website without ever directly handling usernames and password details.  The Password Manager may be preloaded with appropriate passwords and locked, like so;

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new( profile_name => 'locked' ); # using a pre-built profile called 'locked'
    if ($firefox->pwd_mgr_needs_login()) {
        my $new_password = IO::Prompt::prompt(-echo => q[*], 'Enter the password for the locked profile:');
        $firefox->pwd_mgr_login($password);
    } else {
        my $new_password = IO::Prompt::prompt(-echo => q[*], 'Enter the new password for the locked profile:');
        $firefox->pwd_mgr_lock($password);
    }
    ...
    $firefox->pwd_mgr_logout();

Usernames and passwords (for both HTTP Authentication popups and HTML Form based logins) may be added, viewed and deleted.

    use WebService::HIBP();

    my $hibp = WebService::HIBP->new();

    $firefox->add_login(host => 'https://github.com', user => 'me@example.org', password => 'qwerty', user_field => 'login', password_field => 'password');
    $firefox->add_login(host => 'https://pause.perl.org', user => 'AUSER', password => 'qwerty', realm => 'PAUSE');
    ...
    foreach my $login ($firefox->logins()) {
        if ($hibp->password($login->password())) { # does NOT send the password to the HIBP webservice
            warn "HIBP reports that your password for the " . $login->user() " account at " . $login->host() . " has been found in a data breach";
            $firefox->delete_login($login); # how could this possibly help?
        }
    }

And used to fill in login prompts without explicitly knowing the account details.

    $firefox->go('https://pause.perl.org/pause/authenquery')->accept_alert(); # this goes to the page and submits the http auth popup

    $firefox->go('https://github.com/login')->fill_login(); # fill the login and password fields without needing to see them

# REMOTE AUTOMATION OF FIREFOX VIA SSH

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new( host => 'remote.example.org', debug => 1 );
    $firefox->go('https://metacpan.org/');

    # OR specify a different user to login as ...
    
    my $firefox = Firefox::Marionette->new( host => 'remote.example.org', user => 'R2D2', debug => 1 );
    $firefox->go('https://metacpan.org/');

    # OR specify a different port to connect to
    
    my $firefox = Firefox::Marionette->new( host => 'remote.example.org', port => 2222, debug => 1 );
    $firefox->go('https://metacpan.org/');

This module has support for creating and automating an instance of Firefox on a remote node.  It has been tested against a number of operating systems, including recent version of [Windows 10 or Windows Server 2019](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse), OS X, and Linux and BSD distributions.  It expects to be able to login to the remote node via public key authentication.  It can be further secured via the [command](https://man.openbsd.org/sshd#command=_command_) option in the [OpenSSH](https://www.openssh.com/) [authorized\_keys](https://man.openbsd.org/sshd#AUTHORIZED_KEYS_FILE_FORMAT) file such as;

    no-agent-forwarding,no-pty,no-X11-forwarding,permitopen="127.0.0.1:*",command="/usr/local/bin/ssh-auth-cmd-marionette" ssh-rsa AAAA ... == user@server

As an example, the [ssh-auth-cmd-marionette](https://metacpan.org/pod/ssh-auth-cmd-marionette) command is provided as part of this distribution.

When using ssh, Firefox::Marionette will attempt to pass the [TMPDIR](https://en.wikipedia.org/wiki/TMPDIR) environment variable across the ssh connection to make cleanups easier.  In order to allow this, the [AcceptEnv](https://man.openbsd.org/sshd_config#AcceptEnv) setting in the remote [sshd configuration](https://man.openbsd.org/sshd_config) should be set to allow TMPDIR, which will look like;

    AcceptEnv TMPDIR

This module uses [ControlMaster](https://man.openbsd.org/ssh_config#ControlMaster) functionality when using [ssh](https://man.openbsd.org/ssh), for a useful speedup of executing remote commands.  Unfortunately, when using ssh to move from a [cygwin](https://gcc.gnu.org/wiki/SSH_connection_caching), [Windows 10 or Windows Server 2019](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) node to a remote environment, we cannot use [ControlMaster](https://man.openbsd.org/ssh_config#ControlMaster), because at this time, Windows [does not support ControlMaster](https://github.com/Microsoft/vscode-remote-release/issues/96) and therefore this type of automation is still possible, but slower than other client platforms.

# WEBGL

There are a number of steps to getting [WebGL](https://en.wikipedia.org/wiki/WebGL) to work correctly;

- 1. The addons parameter to the [new](https://metacpan.org/pod/Firefox::Marionette#new) method must be set.  This will disable [-safe-mode](http://kb.mozillazine.org/Command_line_arguments#List_of_command_line_arguments_.28incomplete.29)
- 2. The visible parameter to the [new](https://metacpan.org/pod/Firefox::Marionette#new) method must be set.  This is due to [an existing bug in Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1375585).
- 3. [REMOTE AUTOMATION OF FIREFOX VIA SSH](https://metacpan.org/pod/Firefox::Marionette#REMOTE-AUTOMATION-OF-FIREFOX-VIA-SSH) cannot be used with WebGL at the moment.

With all those conditions being met, [WebGL](https://en.wikipedia.org/wiki/WebGL) can be enabled like so;

    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new( addons => 1, visible => 1 );
    if ($firefox->script(q[let c = document.createElement('canvas'); return c.getContext('webgl2') ? true : c.getContext('experimental-webgl') ? true : false;])) {
        $firefox->go("https://get.webgl.org/");
    } else {
        die "WebGL is not supported";
    }

# DIAGNOSTICS

- `Failed to correctly setup the Firefox process`

    The module was unable to retrieve a session id and capabilities from Firefox when it requests a [new\_session](https://metacpan.org/pod/Firefox::Marionette#new_session) as part of the initial setup of the connection to Firefox.

- `Failed to correctly determined the Firefox process id through the initial connection capabilities`

    The module was found that firefox is reporting through it's [Capabilities](https://metacpan.org/pod/Firefox::Marionette::Capabilities#moz_process_id) object a different process id than this module was using.  This is probably a bug in this module's logic.  Please report as described in the BUGS AND LIMITATIONS section below.

- `'%s --version' did not produce output that could be parsed.  Assuming modern Marionette is available:%s`

    The Firefox binary did not produce a version number that could be recognised as a Firefox version number.

- `Failed to create process from '%s':%s`

    The module was to start Firefox process in a Win32 environment.  Something is seriously wrong with your environment.

- `Failed to redirect %s to %s:%s`

    The module was unable to redirect a file handle's output.  Something is seriously wrong with your environment.

- `Failed to exec %s:%s`

    The module was unable to run the Firefox binary.  Check the path is correct and the current user has execute permissions.

- `Failed to fork:%s`

    The module was unable to fork itself, prior to executing a command.  Check the current `ulimit` for max number of user processes.

- `Failed to open directory '%s':%s`

    The module was unable to open a directory.  Something is seriously wrong with your environment.

- `Failed to close directory '%s':%s`

    The module was unable to close a directory.  Something is seriously wrong with your environment.

- `Failed to open '%s' for writing:%s`

    The module was unable to create a file in your temporary directory.  Maybe your disk is full?

- `Failed to open temporary file for writing:%s`

    The module was unable to create a file in your temporary directory.  Maybe your disk is full?

- `Failed to close '%s':%s`

    The module was unable to close a file in your temporary directory.  Maybe your disk is full?

- `Failed to close temporary file:%s`

    The module was unable to close a file in your temporary directory.  Maybe your disk is full?

- `Failed to create temporary directory:%s`

    The module was unable to create a directory in your temporary directory.  Maybe your disk is full?

- `Failed to clear the close-on-exec flag on a temporary file:%s`

    The module was unable to call fcntl using F\_SETFD for a file in your temporary directory.  Something is seriously wrong with your environment.

- `Failed to seek to start of temporary file:%s`

    The module was unable to seek to the start of a file in your temporary directory.  Something is seriously wrong with your environment.

- `Failed to create a socket:%s`

    The module was unable to even create a socket.  Something is seriously wrong with your environment.

- `Failed to connect to %s on port %d:%s`

    The module was unable to connect to the Marionette port.  This is probably a bug in this module's logic.  Please report as described in the BUGS AND LIMITATIONS section below.

- `Firefox killed by a %s signal (%d)`

    Firefox crashed after being hit with a signal.  

- `Firefox exited with a %d`

    Firefox has exited with an error code

- `Failed to bind socket:%s`

    The module was unable to bind a socket to any port.  Something is seriously wrong with your environment.

- `Failed to close random socket:%s`

    The module was unable to close a socket without any reads or writes being performed on it.  Something is seriously wrong with your environment.

- `moz:headless has not been determined correctly`

    The module was unable to correctly determine whether Firefox is running in "headless" or not.  This is probably a bug in this module's logic.  Please report as described in the BUGS AND LIMITATIONS section below.

- `%s method requires a Firefox::Marionette::Element parameter`

    This function was called incorrectly by your code.  Please supply a [Firefox::Marionette::Element](https://metacpan.org/pod/Firefox::Marionette::Element) parameter when calling this function.

- `Failed to write to temporary file:%s`

    The module was unable to write to a file in your temporary directory.  Maybe your disk is full?

- `Failed to close socket to firefox:%s`

    The module was unable to even close a socket.  Something is seriously wrong with your environment.

- `Failed to send request to firefox:%s`

    The module was unable to perform a syswrite on the socket connected to firefox.  Maybe firefox crashed?

- `Failed to read size of response from socket to firefox:%s`

    The module was unable to read from the socket connected to firefox.  Maybe firefox crashed?

- `Failed to read response from socket to firefox:%s`

    The module was unable to read from the socket connected to firefox.  Maybe firefox crashed?

# CONFIGURATION AND ENVIRONMENT

Firefox::Marionette requires no configuration files or environment variables.  It will however use the DISPLAY and XAUTHORITY environment variables to try to connect to an X Server.
It will also use the HTTP\_PROXY, HTTPS\_PROXY, FTP\_PROXY and ALL\_PROXY environment variables as defaults if the session [capabilities](https://metacpan.org/pod/Firefox::Marionette::Capabilities) do not specify proxy information.

# DEPENDENCIES

Firefox::Marionette requires the following non-core Perl modules

- [JSON](https://metacpan.org/pod/JSON)
- [URI](https://metacpan.org/pod/URI)
- [XML::Parser](https://metacpan.org/pod/XML::Parser)
- [Time::Local](https://metacpan.org/pod/Time::Local)

# INCOMPATIBILITIES

None reported.  Always interested in any products with marionette support that this module could be patched to work with.

# BUGS AND LIMITATIONS

## DOWNLOADING USING GO METHOD

When using the [go](https://metacpan.org/pod/Firefox::Marionette#go) method to go directly to a URL containing a downloadable file, Firefox can hang.  You can work around this by setting the [page\_load\_strategy](https://metacpan.org/pod/Firefox::Marionette::Capabilities#page_load_strategy) to `none` like below;

    #! /usr/bin/perl

    use strict;
    use warnings;
    use Firefox::Marionette();

    my $firefox = Firefox::Marionette->new( capabilities => Firefox::Marionette::Capabilities->new( page_load_strategy => 'none' ) );
    $firefox->go("https://github.com/david-dick/firefox-marionette/archive/refs/heads/master.zip");
    while(!$firefox->downloads()) { sleep 1 }
    while($firefox->downloading()) { sleep 1 }
    foreach my $path ($firefox->downloads()) {
        warn "$path has been downloaded";
    }
    $firefox->quit();

## MISSING METHODS

Currently the following Marionette methods have not been implemented;

- WebDriver:SetScreenOrientation

To report a bug, or view the current list of bugs, please visit [https://github.com/david-dick/firefox-marionette/issues](https://github.com/david-dick/firefox-marionette/issues)

# SEE ALSO

- [MozRepl](https://metacpan.org/pod/MozRepl)
- [Selenium::Firefox](https://metacpan.org/pod/Selenium::Firefox)
- [Firefox::Application](https://metacpan.org/pod/Firefox::Application)
- [Mozilla::Mechanize](https://metacpan.org/pod/Mozilla::Mechanize)
- [Gtk2::MozEmbed](https://metacpan.org/pod/Gtk2::MozEmbed)

# AUTHOR

David Dick  `<ddick@cpan.org>`

# ACKNOWLEDGEMENTS

Thanks to the entire Mozilla organisation for a great browser and to the team behind Marionette for providing an interface for automation.

Thanks to [Jan Odvarko](http://www.softwareishard.com/blog/about/) for creating the [HAR Export Trigger](https://github.com/firefox-devtools/har-export-trigger) extension for Firefox.

Thanks to [Mike Kaply](https://mike.kaply.com/about/) for his [post](https://mike.kaply.com/2015/02/10/installing-certificates-into-firefox/) describing importing certificates into Firefox.

Thanks also to the authors of the documentation in the following sources;

- [Marionette Protocol](https://firefox-source-docs.mozilla.org/testing/marionette/marionette/index.html)
- [Marionette Documentation](https://firefox-source-docs.mozilla.org/testing/marionette/marionette/index.html)
- [Marionette driver.js](https://hg.mozilla.org/mozilla-central/file/tip/remote/marionette/driver.js)
- [about:config](http://kb.mozillazine.org/About:config_entries)
- [nsIPrefService interface](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Reference/Interface/nsIPrefService)

# LICENSE AND COPYRIGHT

Copyright (c) 2021, David Dick `<ddick@cpan.org>`. All rights reserved.

This module is free software; you can redistribute it and/or
modify it under the same terms as Perl itself. See ["perlartistic" in perlartistic](https://metacpan.org/pod/perlartistic#perlartistic).

The [Firefox::Marionette::Extension::HarExportTrigger](https://metacpan.org/pod/Firefox::Marionette::Extension::HarExportTrigger) module includes the [HAR Export Trigger](https://github.com/firefox-devtools/har-export-trigger)
extension which is licensed under the [Mozilla Public License 2.0](https://www.mozilla.org/en-US/MPL/2.0/).

# DISCLAIMER OF WARRANTY

BECAUSE THIS SOFTWARE IS LICENSED FREE OF CHARGE, THERE IS NO WARRANTY
FOR THE SOFTWARE, TO THE EXTENT PERMITTED BY APPLICABLE LAW. EXCEPT WHEN
OTHERWISE STATED IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES
PROVIDE THE SOFTWARE "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER
EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE
ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE SOFTWARE IS WITH
YOU. SHOULD THE SOFTWARE PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL
NECESSARY SERVICING, REPAIR, OR CORRECTION.

IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING
WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MAY MODIFY AND/OR
REDISTRIBUTE THE SOFTWARE AS PERMITTED BY THE ABOVE LICENCE, BE
LIABLE TO YOU FOR DAMAGES, INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL,
OR CONSEQUENTIAL DAMAGES ARISING OUT OF THE USE OR INABILITY TO USE
THE SOFTWARE (INCLUDING BUT NOT LIMITED TO LOSS OF DATA OR DATA BEING
RENDERED INACCURATE OR LOSSES SUSTAINED BY YOU OR THIRD PARTIES OR A
FAILURE OF THE SOFTWARE TO OPERATE WITH ANY OTHER SOFTWARE), EVEN IF
SUCH HOLDER OR OTHER PARTY HAS BEEN ADVISED OF THE POSSIBILITY OF
SUCH DAMAGES.
