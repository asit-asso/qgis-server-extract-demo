# Projet de test/démo Extract <-> QGIS Server

## Objectif

Démonter/tester l'interopérabilité entre QGIS Server et Extract pour l'impression d'un Atlas, basé sur une emprise géographique pour filtrer les pages de l'Atlas.

## Contenu
- Dockerfile QGIS Server
- Config NGINX
- Projet QGIS de démo avec un layer de donnée et un layer de coverage pour l'Atlas (découpage)

## Utilisation

```sh
docker-compose up
```

ou 

```sh
docker compose up
```

Une fois les images buildées et les containers UP vous pourrez accédez aux services QGIS Server : `http://127.0.0.1:8888/qgis-server`

## Config Extract pour utiliser ce projet

![Screenshot config Extract](/doc/extract_config.png)

## Interroger les services pour l'impression

La chaîne est la suivante : `GetProjectSettings` -> `GetFeature` -> `GetPrint`

Obtenir la définition des Atlas : `http://127.0.0.1:8888/qgis-server?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetProjectSettings`

On trouve la couche de coverage avec ce Xpath : ` /WMS_Capabilities/Capability/ComposerTemplates/ComposerTemplate[@name='myplan']/@atlasCoverageLayer`

On peut lister les features de la couche de coverage qui intersectent une zone :
`POST http://127.0.0.1:8888/qgis-server?`

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<wfs:GetFeature maxFeatures="5000" version="1.1.0" service="WFS" xsi:schemaLocation="http://www.opengis.net/wfs http://schemas.opengis.net/wfs/1.1.0/wfs.xsd" xmlns:wfs="http://www.opengis.net/wfs" xmlns:gml="http://www.opengis.net/gml" xmlns:ogc="http://www.opengis.net/ogc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <wfs:Query typeName="GeoCoverDivision">
    <ogc:Filter>
      <ogc:Intersects>
        <ogc:PropertyName>the_geom</ogc:PropertyName>
          <Polygon xmlns="http://www.opengis.net/gml" srsName="EPSG:4326">
            <exterior>
              <LinearRing>
                <posList srsDimension="2">6.904809316884131 46.445147801931604 6.904511260615304 46.44536023617469 6.905209363082197 46.44564162252219 6.905951438924574 46.445877208874904 6.906586641229295 46.44633044643151 6.907835703094311 46.44689045221654 6.908457065788995 46.44696353109655 6.908648245758518 46.44684388210399 6.907904997673016 46.446725253908994 6.906745292077869 46.446141820152185 6.906105531914959 46.445749169409744 6.905239061797321 46.44545902316597 6.904809316884131 46.445147801931604</posList>
              </LinearRing>
            </exterior>
          </Polygon>
      </ogc:Intersects>
    </ogc:Filter>
  </wfs:Query>
</wfs:GetFeature>
```

On peut extraire les IDs des features (xpath ci dessous): :
`/wfs:FeatureCollection/gml:featureMember/qgs:MY_COVERAGE_LAYER/@gml:id`

Avec les IDs retournés, on peut lancer un print :
`http://127.0.0.1:8888/qgis-server?SERVICE=WMS&REQUEST=GetPrint&CRS=EPSG:2056&TEMPLATE=mybeautifullayout&FORMAT=pdf&ATLAS_PK=119`


## Sources

- Données du projet QGIS
  - Découpage GeoCover : https://opendata.swiss/fr/dataset/blatteinteilung-geologische-vektordatensatze-geocover (OGD : Utilisation libre. Obligation d’indiquer la source.)
  - Vector tiles Swisstopo Light Base Map : https://www.swisstopo.admin.ch/fr/cartes-web-light-base-map ch.swisstopo.leichte-basiskarte.vt : (OGD + © MapTiler, © OpenStreetMap contributors)
- Dockerfile QGIS Server : https://docs.qgis.org/3.34/en/docs/server_manual/containerized_deployment.html#simple-docker-images (Licence GNU de documentation libre, © QGIS project