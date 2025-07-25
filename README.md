# Curb - Libcurl bindings for Ruby

[![CI](https://github.com/taf2/curb/actions/workflows/ci.yml/badge.svg)](https://github.com/taf2/curb/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/taf2/curb/branch/master/graph/badge.svg)](https://codecov.io/gh/taf2/curb)
[![Gem Version](https://badge.fury.io/rb/curb.svg)](https://badge.fury.io/rb/curb)

* [CI Build Status](https://github.com/taf2/curb/actions/workflows/ci.yml)
* [rubydoc rdoc](http://www.rubydoc.info/github/taf2/curb/)
* [github project](http://github.com/taf2/curb/tree/master)

Curb (probably CUrl-RuBy or something) provides Ruby-language bindings for the
libcurl(3), a fully-featured client-side URL transfer library.
cURL and libcurl live at [https://curl.se/libcurl/](https://curl.se/libcurl/) .

Curb is a work-in-progress, and currently only supports libcurl's `easy` and `multi` modes.

A big advantage to Curb over all other known ruby http libraries is it's ability to handle timeouts without the use of threads.

## License

Curb is copyright (c) 2006 Ross Bamford, and released under the terms of the
Ruby license. See the LICENSE file for the gory details.

## Easy mode

GET request
```
  res = Curl.get("https://www.google.com/") {|http|
    http.timeout = 10 # raise exception if request/response not handled within 10 seconds
  }
  puts res.code
  puts res.head
  puts res.body
```

POST request
```
  res = Curl.post("https://your-server.com/endpoint", {post: "this"}.to_json) {|http|
    http.headers["Content-Type"] = "application/json"
  }
  puts res.code
  puts res.head
  puts res.body
```

## FTP Support

require 'curb'

### Basic FTP Download
```ruby
puts "=== FTP Download Example ==="
ftp = Curl::Easy.new('ftp://ftp.example.com/remote/file.txt')
ftp.username = 'user'
ftp.password = 'password'
ftp.perform
puts ftp.body
```

### FTP Upload
```ruby
puts "\n=== FTP Upload Example ==="
upload = Curl::Easy.new('ftp://ftp.example.com/remote/upload.txt')
upload.username = 'user'
upload.password = 'password'
upload.upload = true
upload.put_data = File.read('local_file.txt')
upload.perform
```

### List Directory Contents
```ruby
puts "\n=== FTP Directory Listing Example ==="
list = Curl::Easy.new('ftp://ftp.example.com/remote/directory/')
list.username = 'user'
list.password = 'password'
list.dirlistonly = true
list.perform
puts list.body
```

### Advanced FTP Usage with Various Options
```
puts "\n=== Advanced FTP Example ==="
advanced = Curl::Easy.new do |curl|
  curl.url = 'ftp://ftp.example.com/remote/file.txt'
  curl.username = 'user'
  curl.password = 'password'

  # FTP Options
  curl.ftp_response_timeout = 30
  curl.ftp_create_missing_dirs = true   # Create directories if they don't exist
  curl.ftp_filemethod = Curl::CURL_MULTICWD  # Use multicwd method for traversing paths

  # SSL/TLS Options for FTPS
  curl.use_ssl = Curl::CURLUSESSL_ALL  # Use SSL/TLS for control and data
  curl.ssl_verify_peer = true
  curl.ssl_verify_host = true
  curl.cacert = "/path/to/cacert.pem"

  # Progress callback
  curl.on_progress do |dl_total, dl_now, ul_total, ul_now|
    puts "Download: #{dl_now}/#{dl_total} Upload: #{ul_now}/#{ul_total}"
    true # must return true to continue
  end

  # Debug output
  curl.verbose = true
  curl.on_debug do |type, data|
    puts "#{type}: #{data}"
    true
  end
end

advanced.perform
```

### Parallel FTP Downloads
```
puts "\n=== Parallel FTP Downloads Example ==="
urls = [
  'ftp://ftp.example.com/file1.txt',
  'ftp://ftp.example.com/file2.txt',
  'ftp://ftp.example.com/file3.txt'
]
```

### Common options for all connections
```
options = {
  :username => 'user',
  :password => 'password',
  :timeout => 30,
  :on_success => proc { |easy| puts "Successfully downloaded: #{easy.url}" },
  :on_failure => proc { |easy, code| puts "Failed to download: #{easy.url} (#{code})" }
}

Curl::Multi.download(urls, options) do |curl, file_path|
  puts "Completed downloading to: #{file_path}"
end
```

## You will need

* A working Ruby installation (`2.0.0+` will work but `2.1+` preferred) (it's possible it still works with 1.8.7 but you'd have to tell me if not...)
* A working libcurl development installation
(Ideally one of the versions listed in the compatibility chart below that maps to your `curb` version)
* A sane build environment (e.g. gcc, make)

## Version Compatibility chart

A **non-exhaustive** set of compatibility versions of the libcurl library
with this gem are as follows. (Note that these are only the ones that have been
tested and reported to work across a variety of platforms / rubies)

| Gem Version | Release Date   | libcurl versions  |
| ----------- | -------------- | ----------------- |
| 1.0.8       | Feb 10, 2025   | 7.58 – 8.12.1     |
| 1.0.7       | Feb 09, 2025   | 7.58 – 8.12.1     |
| 1.0.6       | Aug 23, 2024   | 7.58 – 8.12.1     |
| 1.0.5       | Jan 2023       | 7.58 – 8.12.1     |
| 1.0.4       | Jan 2023       | 7.58 – 8.12.1     |
| 1.0.3*      | Dec 2022       | 7.58 – 8.12.1     |
| 1.0.2*      | Dec 2022       | 7.58 – 8.12.1     |
| 1.0.1       | Apr 2022       | 7.58 – 8.12.1     |
| 1.0.0       | Jan 2022       | 7.58 – 8.12.1     |
| 0.9.8       | Jan 2019       | 7.58 – 7.81       |
| 0.9.7       | Nov 2018       | 7.56 – 7.60       |
| 0.9.6       | May 2018       | 7.51 – 7.59       |
| 0.9.5       | May 2018       | 7.51 – 7.59       |
| 0.9.4       | Aug 2017       | 7.41 – 7.58       |
| 0.9.3       | Apr 2016       | 7.26 – 7.58       |

  ```*avoid using these version are known to have issues with segmentation faults```

## Installation...

... will usually be as simple as:

    $ gem install curb

On Windows, make sure you're using the [DevKit](http://rubyinstaller.org/downloads/) and
the [development version of libcurl](http://curl.se/gknw.net/7.39.0/dist-w32/curl-7.39.0-devel-mingw32.zip). Unzip, then run this in your command
line (alter paths to your curl location, but remember to use forward slashes):

    gem install curb --platform=ruby -- --with-curl-lib=C:/curl-7.39.0-devel-mingw32/lib --with-curl-include=C:/curl-7.39.0-devel-mingw32/include
    
Note that with Windows moving from one method of compiling to another as of Ruby `2.4` (DevKit -> MYSYS2),
the usage of Ruby `2.4+` with this gem on windows is unlikely to work. It is advised to use the
latest version of Ruby 2.3 available [HERE](https://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.3.3.exe)

Or, if you downloaded the archive:

    $ rake compile && rake install

If you have a weird setup, you might need extconf options. In this case, pass
them like so:

    $ rake compile EXTCONF_OPTS='--with-curl-dir=/path/to/libcurl --prefix=/what/ever' && rake install

Curb is tested only on GNU/Linux x86 and Mac OSX - YMMV on other platforms.
If you do use another platform and experience problems, or if you can
expand on the above instructions, please report the issue at http://github.com/taf2/curb/issues

On Ubuntu, the dependencies can be satisfied by installing the following packages:

18.04 and onwards
    
    $ sudo apt-get install libcurl4 libcurl3-gnutls libcurl4-openssl-dev

< 18.04

    $ sudo apt-get install libcurl3 libcurl3-gnutls libcurl4-openssl-dev

On RedHat:

    $ sudo yum install ruby-devel libcurl-devel openssl-devel

Curb has fairly extensive RDoc comments in the source. You can build the
documentation with:

    $ rake doc

## Usage & examples

Curb provides two classes:

* `Curl::Easy` - simple API, for day-to-day tasks.
* `Curl::Multi` - more advanced API, for operating on multiple URLs simultaneously.

To use either, you will need to require the curb gem:

```ruby
require 'curb'
```

### Super simple API (less typing)

```ruby
http = Curl.get("http://www.google.com/")
puts http.body

http = Curl.post("http://www.google.com/", {:foo => "bar"})
puts http.body

http = Curl.get("http://www.google.com/") do |http|
  http.headers['Cookie'] = 'foo=1;bar=2'
end
puts http.body
```

### Simple fetch via HTTP:

```ruby
c = Curl::Easy.perform("http://www.google.co.uk")
puts c.body
```

Same thing, more manual:

```ruby
c = Curl::Easy.new("http://www.google.co.uk")
c.perform
puts c.body
```

### Additional config:

```ruby
http = Curl::Easy.perform("http://www.google.co.uk") do |curl|
  curl.headers["User-Agent"] = "myapp-0.0"
  curl.verbose = true
end
```

Same thing, more manual:

```ruby
c = Curl::Easy.new("http://www.google.co.uk") do |curl|
  curl.headers["User-Agent"] = "myapp-0.0"
  curl.verbose = true
end

c.perform
```

### HTTP basic authentication:

```ruby
c = Curl::Easy.new("http://github.com/")
c.http_auth_types = :basic
c.username = 'foo'
c.password = 'bar'
c.perform
```

### HTTP "insecure" SSL connections (like curl -k, --insecure) to avoid Curl::Err::SSLCACertificateError:

```ruby
c = Curl::Easy.new("https://github.com/")
c.ssl_verify_peer = false
c.perform
```

### Supplying custom handlers:

```ruby
c = Curl::Easy.new("http://www.google.co.uk")

c.on_body { |data| print(data) }
c.on_header { |data| print(data) }

c.perform
```

### Reusing Curls:

```ruby
c = Curl::Easy.new

["http://www.google.co.uk", "http://www.ruby-lang.org/"].map do |url|
  c.url = url
  c.perform
  c.body
end
```

### HTTP POST form:

```ruby
c = Curl::Easy.http_post("http://my.rails.box/thing/create",
                         Curl::PostField.content('thing[name]', 'box'),
                         Curl::PostField.content('thing[type]', 'storage'))
```

### HTTP POST file upload:

```ruby
c = Curl::Easy.new("http://my.rails.box/files/upload")
c.multipart_form_post = true
c.http_post(Curl::PostField.file('thing[file]', 'myfile.rb'))
```

### Using HTTP/2

```ruby
c = Curl::Easy.new("https://http2.akamai.com")
c.set(:HTTP_VERSION, Curl::HTTP_2_0)

c.perform
puts (c.body.include? "You are using HTTP/2 right now!") ? "HTTP/2" : "HTTP/1.x"
```

### Multi Interface (Basic HTTP GET):

```ruby
# make multiple GET requests
easy_options = {:follow_location => true}
# Use Curl::CURLPIPE_MULTIPLEX for HTTP/2 multiplexing
multi_options = {:pipeline => Curl::CURLPIPE_HTTP1}

Curl::Multi.get(['url1','url2','url3','url4','url5'], easy_options, multi_options) do|easy|
  # do something interesting with the easy response
  puts easy.last_effective_url
end
```

### Multi Interface (Basic HTTP POST):

```ruby
# make multiple POST requests
easy_options = {:follow_location => true, :multipart_form_post => true}
multi_options = {:pipeline => Curl::CURLPIPE_HTTP1}


url_fields = [
  { :url => 'url1', :post_fields => {'f1' => 'v1'} },
  { :url => 'url2', :post_fields => {'f1' => 'v1'} },
  { :url => 'url3', :post_fields => {'f1' => 'v1'} }
]

Curl::Multi.post(url_fields, easy_options, multi_options) do|easy|
  # do something interesting with the easy response
  puts easy.last_effective_url
end
```

### Multi Interface (Advanced):

```ruby
responses = {}
requests = ["http://www.google.co.uk/", "http://www.ruby-lang.org/"]
m = Curl::Multi.new
# add a few easy handles
requests.each do |url|
  responses[url] = ""
  c = Curl::Easy.new(url) do|curl|
    curl.follow_location = true
    curl.on_body{|data| responses[url] << data; data.size }
    curl.on_success {|easy| puts "success, add more easy handles" }
  end
  m.add(c)
end

m.perform do
  puts "idling... can do some work here"
end

requests.each do|url|
  puts responses[url]
end
```

### Easy Callbacks

* `on_success`  is called when the response code is 2xx
* `on_redirect` is called when the response code is 3xx
* `on_missing` is called when the response code is 4xx
* `on_failure` is called when the response code is 5xx
* `on_complete` is called in all cases.
