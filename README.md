# 99 bottles of beer lyrics in Apache Sling

proposal for a candidate of http://99-bottles-of-beer.net leveraging:
- Apache Sling Pipes (https://sling.apache.org/documentation/bundles/sling-pipes.html),
- Apache Sling Resource resolution (https://sling.apache.org/documentation/the-sling-engine/resources.html),
- Apache Sling HTL (https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html),
- Apache Felix Gogo (http://felix.apache.org/documentation/subprojects/apache-felix-gogo.html)
- Apache Sling Query (https://sling.apache.org/documentation/bundles/sling-query.html)

`mvn clean install content-package`

in gogo shell, run the following command that parses the website and creates /var/bottles tree
```run egrep 'http://99-bottles-of-beer.net/lyrics.html' @ name bottles @ with 'pattern=(?<number>\d(\d)?) bottle(s)? of beer on the wall,' / mkdir '/var/bottles/${bottles.number}' / write 'onTheWall=${bottles.number}' 'offTheWall=${Number(bottles.number)-1}' 'sling:resourceType=bottles/line'``

in your favourite groovy console, run the following groovy script, build a list pipe like this one:
```
def plumber = getService("org.apache.sling.pipes.Plumber");

plumber.newPipe(resourceResolver)
  .echo("/var")
  .children('sling:Folder#bottles')
  .$('[sling:resourceType=bottles/line]').name("line")
  .outputs("On the Wall!",'${line.onTheWall}')
  .build("/services/adapt/list");
  ```
  
 check that list pipe is working my running following queries:

`GET /services/adapt/list.csv?size=99`
`GET /services/adapt/list.json`

now, back in gogo, add a subnode to describe the song component how to search for lines

`run mkdir /var/bottles/pipes/lines / write "sling:resourceType=slingPipes/reference" "expr=/services/adapt/list"`


still in gogo run that command that will set resource type of the song, allow access to user imccoy, and package up bottles for safety

`run echo /var/bottles / write 'sling:resourceType=bottles/song' / allow imccoy / pkg /etc/packages/bottles-backup.zip`


Now enjoy the lyrics of the song by accessing

`GET /var/bottles.html`
