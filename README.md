# Frontback

Pluggable front-end issue submission for VCS and issue boards.  A script allows end users to take screenshots of a problem, add a comment and automatic browser info, and submit it all as an issue or card.

This leans heavily on:
- [ivoviz/feedback](https://github.com/ivoviz/feedback)
- [niklasvh/html2canvas](https://github.com/niklasvh/html2canvas)

## Currently supporting

- Gitlab
- Trello

## Quickstart

### Add the plugin to a page:

```html
<script>
	window.frontback = {
		repo: 'https://gitlab.com/newcity/test',
		postUrl: 'http://' + host + ':9000'
	};
	var script = document.createElement('script');
	script.src = frontback.postUrl + '/assets/js/frontback.js';
	document.body.appendChild(script);
</script>
```

Where the first two variables define the homepage of the repository and the endpoint of the proxy.

### Configure the service targets at the endpoint

In a place accessible to the endpoint proxy, add configuration in a json array. Services may be combined in a single file and APIs are selected based on the homepage URL.

### Gitlab

Find your Gitlab private token from https://gitlab.com/profile/personal_access_tokens (or similar for your hosted instance)

Use the project homepage as a key:

```json
{
    "https://gitlab.com/newcity/test": {
        "private_token": "GITLAB-PRIVATE-TOKEN",
        "assignee_id": "ahebrank"
    },
		...
}
```

### Trello

Generate a Trello application key at https://trello.com/app-key and use that to generate an auth token from https://trello.com/1/connect?key=[key]&name=Frontback&response_type=token&scope=read,write

Then use the trello board URL as the key:

```json
{
		"https://trello.com/b/S1QWR14x/api-test": {
				"app_key": "APPLICATION_KEY",
				"private_token": "PRIVATE_AUTH_TOKEN",
				"assignee_id": "ahebrank"
		}
}
```

### Start it up

The python wsgi web stack configuration has a lot of pieces in its newer incarnation.
The following skips over setting up a virtual environment and assumes the flask app
is installed in `/usr/local/frontback/endpoint`.

#### Newer way: Nginx uwsgi proxy

1. Make sure `uwsgi` is installed (e.g., `apt-get install uwsgi`)
2. Copy the upstart file to `/etc/init` (`cp /usr/local/frontback/endpoint/frontback.conf.upstart /etc/init/frontback.config`)
3. Start it up (e.g., `service frontback start`)
4. (optionally) Make it persistent (e.g., `initctl reload-configuration`)
5. Make sure the uwsgi parameters are availble to nginx
6. Add to a `server` block in nginx configuration:

```
location /frontback {
   uwsgi_pass unix:/usr/local/frontback/endpoint/frontback.sock;
   include /etc/nginx/uwsgi_params;
   uwsgi_param UWSGI_SCRIPT /frontback;
}
```

See e.g., https://www.digitalocean.com/community/tutorials/how-to-deploy-python-wsgi-applications-using-uwsgi-web-server-with-nginx for more detail about setting up and proxying uwsgi applications.

#### Older way: Apache2 with mod_wsgi

1. Install mod_wsgi (e.g., `apt-get install libapache2-mod-wsgi`)
2. Within an apache2 virtual host, add config like:

```
WSGIDaemonProcess frontback user=www-data group=www-data threads=5 home=
/usr/local/frontback/endpoint
WSGIScriptAlias / /usr/local/frontback/endpoint/wsgi.py

<Directory /usr/local/frontback/endpoint>
  	WSGIProcessGroup frontback
  	WSGIApplicationGroup %{GLOBAL}
		Require all granted
</Directory>
```


## Building and testing

`gulp` builds styles and scripts and puts up a test page at `http://localhost:3000`.  See `test/index.html` to modify the plugin configuration.

You can run the flask endpoint locally with:

```bash
cd endpoint
python issue_proxy.py -c repos.json -p 9000
```
