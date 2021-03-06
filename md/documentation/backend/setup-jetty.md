# Jetty 8.1.16 with pre-installed/configured Oskari

After this you will have Oskari running including 

- Oskari frontend code (https://github.com/nls-oskari/oskari)
- Oskari server (map functionality: https://github.com/nls-oskari/oskari-server/tree/master/webapp-map)
- Oskari transport (WFS services: https://github.com/nls-oskari/oskari-server/tree/master/webapp-transport)
- Oskari printout (Print services: https://github.com/nls-oskari/oskari-server/tree/master/servlet-printout)
- Geoserver 2.7.1.1 with WPS-plugin and Oskari extensions (https://github.com/nls-oskari/oskari-server/tree/master/geoserver-ext)

### Requirements

* JDK 1.7+ (tested with Oracle Java 1.7.0_51 and 1.8.0_05, should run on OpenJDK as well)
* Database available: [Instructions for setting up database](/documentation/backend/setup-database)
* Redis (Optional, required for WFS and print functionalities): [Setup Redis](/documentation/backend/setup-redis)

## Setting up Jetty

1) Download the [Jetty Bundle](/download)

2) Unpack the zip file to selected location

The zip includes README.txt, KnownIssues.txt and the Jetty folder (referred as `{JETTY_HOME}`)

3) Configure the database properties (host/credentials) by editing `{JETTY_HOME}/resources/oskari-ext.properties`

    db.url=jdbc:postgresql://[host]:[port]/[dbname]
    db.username=[user]
    db.password=[passwd]

4) Startup the Jetty by running (in `{JETTY_HOME}`)

    java -jar start.jar

5) After Jetty is up and running open a browser with URL

    http://localhost:8080


You can login as:
- user with username "user" and password "user" 
- admin with username "admin" and password "oskari"

---

## Defaults/assumptions

The preconfigured Jetty uses these defaults. These can be changed by modifying `{JETTY_HOME}/resources/oskari-ext.properties`.

Redis:
- redis running on localhost at default port (6379)

Database (Postgres with postgis extension)
- db URL: localhost in default port (5432)
- db name: oskaridb
- db user: oskari/oskari

Geoserver (provided in jetty bundle)
- url: http://localhost:8080/geoserver
- user: admin/geoserver
- datadir: {JETTY-HOME}/geoserver_data (configurable in {JETTY-HOME}/start.ini)
- if local geoserver content doesn't seem to work correctly (log shows "feature not on screen" or SRID errors) -> try logging into geoserver and reload the feature type under layers (my_places_categories, user_layer_data_style, analysis_data_style). This is probably due to geoserver starting before Oskari has created the database. We are exploring the option to configure Geoserver through it's REST API to workaround this and preconfigured datadir.

Oskari (provided in jetty bundle)
- url: http://localhost:8080/

## Custom configurations

### Removing the unnecessary parts

Oskari-server can run with just the oskari-map webapp. If you don't need all the features, you can remove them from under `{JETTY_HOME}/webapps`:
- user content functionalities: you can remove `geoserver` folder
- WFS functionalities: you can remove `transport.war` file
- Print functionality: you can remove `oskari-printout-backend.war` file

You will also need to remove the corresponding parts of the UI so users don't have access to them. This is done by removing bundles from views and currently it needs to be done by modifying the database content. Bundles are linked to views in the database table `portti_view_bundle_seq` and functionalities are removed from the UI by deleting rows from the table.

### Editing article content

- User guide: edit the file in {JETTY-HOME}/resources/articlesByTag/userguide.html
- Publisher terms of use: edit the file in {JETTY-HOME}/resources/articlesByTag/termsofuse__mappublication__en.html

### Changing the default port

- change in `{JETTY_HOME}/etc/jetty.xml`
    
    <Call name="addConnector">
      <Arg>
          <New class="org.eclipse.jetty.server.nio.SelectChannelConnector">
            <Set name="port"><Property name="jetty.port" default="8080"/></Set>

- change `{JETTY_HOME}/resources/oskari-ext.properties` where ever `8080` is referenced 
- change `{JETTY_HOME}/resources/transport-ext.properties` where ever `8080` is referenced 
- check the "Using external Geoserver" below (also refers to localhost:8080 port)

### Proxy settings

If you need a proxy to access internet you can configure it in `{JETTY_HOME}/start.ini`

	-Dhttp.proxyHost=
	-Dhttp.proxyPort=
	-Dhttp.nonProxyHosts=
	-Dhttps.proxyHost=
	-Dhttps.proxyPort=
	-Dhttps.nonProxyHosts=

### Database url/name/user/pass are changed
`{JETTY_HOME}/resources/oskari-ext.properties` needs to be updated

	db.url=jdbc:postgresql://[host]:[port]/[dbname]
	db.username=[user]
	db.password=[passwd]

Stores in geoserver needs to be updated and re-enabled for myplaces/analysis/userlayers to work

### Using external Geoserver
- `{JETTY_HOME}/resources/oskari-ext.properties` needs to be updated (multiple geoserver references)
- layers pointing to local geoserver in database needs to be updated (table: oskari_maplayer - columns: url, username and password)

### Using external Redis
`{JETTY_HOME}/resources/oskari-ext.properties` needs to be updated 

	redis.hostname=localhost
	redis.port=6379
	redis.pool.size=10

### How the Jetty bundle was built

See [here](/documentation/backend/creating-jetty-bundle) for details