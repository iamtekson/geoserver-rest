### Installation
The `geoserver-rest` package is useful for the management for geospatial data in [GeoServer](http://geoserver.org/). The package is useful for the creating, updating and deleting geoserver workspaces, stores, layers, and style files. For installation of this package, following packages should be installed first:

###### Dependencies
1. [GDAL](https://gdal.org/) 
2. [Pycurl](http://pycurl.io/)

The `geoserver-rest` package can be installed with pip, if all the dependencies are installed already.

```bash
pip install geoserver-rest
```

### How to use
This library is used for creating workspace, coveragestore, featurestore, styles. Some of the examples are shown below.

##### Initialize the library
This step is used to initialize the library. It takes parameters as geoserver url, username, password.

```python
from geo.Geoserver import Geoserver
geo = Geoserver('http://localhost:8080/geoserver', username='admin', password='geoserver')
```

##### Create workspace
```python
geo.create_workspace(workspace='demo')
```

##### Create coveragestore
It is helpful for publishing the **raster data** to the geoserver. Here if you don't pass the `lyr_name` parameter, it will take the raster file name as the layer name. 

```python
geo.create_coveragestore(lyr_name='layer1' path=r'path\to\raster\file.tif', workspace='demo')
```

If the layername already exists in geoserver, you can pass another parameter `overwrite=True`,

```python
geo.create_coveragestore(lyr_name='layer1' path=r'path\to\raster\file.tif', workspace='demo' overwrite=True)
```

##### Create featurestore and publish layer
It is used for connecting the **PostGIS** with geoserver and publish this as a layer. It is only useful for **vector data**. The postgres connection parameters should be passed as the parameters. For publishing the PostGIS tables, the pg_table parameter represent the `table name in postgres`

```python
geo.create_featurestore(store_name='geo_data', workspace='demo', db='postgres', host='localhost', pg_user='postgres', pg_password='admin')
geo.publish_featurestore(workspace='demo', store_name='geo_data', pg_table='geodata_table_name')
```

##### Upload style and publish it
It is used for uploading **SLD** files and publish style. If the style name already exists, you can pass the parameter `overwrite=True` to overwrite it. The name of the style will be name of the uploaded file name.

```python
geo.upload_style(path=r'path\to\sld\file.sld', workspace='demo')
geo.publish_style(layer_name='geoserver_layer_name', style_name='sld_file_name', workspace='demo')
```


##### Create Coverage Style based on the raster (Dynamic) and apply style
It is used to create the style file for **raster data**. You can get the `color_ramp` name from [matplotlib colormaps](https://matplotlib.org/3.3.0/tutorials/colors/colormaps.html). By default `color_ramp='RdYlGn'` (red to green color ramp).

```python
#Style name will be the same as the raster_file_name
geo.create_coveragestyle(raster_path=r'path\to\raster\file.tiff', style_name='style_1', workspace='demo', color_ramp='RdYiGn')
geo.publish_style(layer_name='geoserver_layer_name', style_name='raster_file_name', workspace='demo')
```

For generating the style for classified raster, you can pass the another parameter called `cmap_type='values'` as,

```python
geo.create_coveragestyle(raster_path=r'path\to\raster\file.tiff', style_name='style_1', workspace='demo', color_ramp='RdYiGn', cmap_type='values')
```

| Option      | Type    | Default   | Description                                                                                                                                     |
| ----------- | ------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| style_name      | string  | file_name | This is optional field. If you don't pass the style_name parameter, then it will take the raster file name as the default name of style in geoserver          |
| raster_path      | path  | None | path to the raster file                                                            |
| workspace | string | None      | The name of the workspace                                                                                                                          |
| color_ramp  | string  | RdYlGn   | The color ramp name. The name of the color ramp can be found here in [matplotlib colormaps](https://matplotlib.org/3.3.0/tutorials/colors/colormaps.html) |
| cmap_type      | string | ramp     | By default the continuous style will be generated, If you want to generate the style for classified raster then pass the parameter `color_ramp='values'`                                                                                        |
| overwrite | boolean  | False     | For overwriting the previous style file in geoserver                              |


##### Create featur style
It is used for creating the style for point, line and polygon dynamically. Currently, it supports three different types of feature styles,

1. Outline featurestyle: For creating the style which have only boundary color but not the fill style
2. Catagorized featurestyle: For creating catagorized dataset
3. Classified featurestyle: Classify the input data and style it: (For now, it only supports polygon geometry)

```python
geo.create_outline_featurestyle(style_name='new_style' color="#3579b1" geom_type='polygon', workspace='demo')
geo.create_catagorized_featurestyle(style_name='name_of_style', column_name='name_of_column', column_distinct_values=[1,2,3,4,5,6,7], workspace='demo')
geo.create_classified_featurestyle(style_name='name_of_style' column_name='name_of_column', column_distinct_values=[1,2,3,4,5,6,7], workspace='demo')
```

**Note:** 
* The geom_type must be either`point`,`line` or `polygon`. 
* The `color_ramp` name can be obtained from [matplotlib colormaps](https://matplotlib.org/3.3.0/tutorials/colors/colormaps.html).

###### The options for creating catagorized/classified featurestyle are as follows,
| Option      | Type    | Default   | Description                                                                                                                                     |
| ----------- | ------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| style_name      | string  | None | The name of the style file in geoserver         |
| column_name      | string  | None | The name of the column, based on which the style will be generated                                                   |
| column_distinct_values | list/array | None      | The column distinct values based on which the style will be applied/classified                                                                                                                          |
|workspace | string | None | The name of the workspace
| color_ramp  | string  | RdYlGn   | The color ramp name. The name of the color ramp can be found here in [matplotlib colormaps](https://matplotlib.org/3.3.0/tutorials/colors/colormaps.html) |
| geom_type      | string | polygon     | The geometry type, available options are `point`, `line` or `polygon`               |
| outline_color | color hex value | '#3579b1' | The outline color of the polygon/line |
| overwrite | boolean  | False     | For overwriting the previous style file in geoserver                              |

##### Some of the delete request examples
```python
# delete workspace 
geo.delete_workspace(workspace='demo')

# delete layer
geo.delete_layer(layer_name='agri_final_proj', workspace='demo')

# delete feature store, i.e. remove postgresql connection
geo.delete_featurestore(featurestore_name='ftry', workspace='demo')

# delete coveragestore, i.e. delete raster store
geo.delete_coveragestore(coveragestore_name='agri_final_proj', workspace='demo')

# delete style file
geo.delete_style(style_name='kamal2', workspace='demo')
```

##### Automation process
The following code will first convert all the `.rst` data format inside `C:\Users\gic\Desktop\etlIa\` folder, into `tiff` format and then upload all the `tiff` files to the GeoServer.

```python
from geo.Geoserver import Geoserver
from osgeo import gdal
import glob
import os

geo = Geoserver('http://localhost:8080/geoserver', username='admin', password='geoserver')

rst_files = glob.glob(r'C:\Users\gic\Desktop\etlIa\*.rst')
# geo.create_workspace('geonode')

for rst in rst_files:
    file_name = os.path.basename(file_name)
    src = gdal.Open(rst)
    tiff = r'C:\Users\tek\Desktop\try\{}'.format(file_name)
    gdal.Translate(tiff, src)
    geo.create_coveragestore(lyr_name=file_name, path=tiff, workspace='geonode')    #, overwrite=True
```

### Acknowledgements
Created and managed by [Tek Bahadur Kshetri](http://tekkshetri.com.np/) for the activites of Geoinformatics Center of Asian Institute of Technology, Thailand.


### Appreciate:
If you liked my work, show some :heart: by :star: repo.

Also you can appreciate by

<p>
 <table style="border-spacing: 5px 10px;">

 <tr>
  <td>
<a href="https://www.buymeacoffee.com/iamtekson"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" style="max-width:90%;" width="200" height="60"></a>
</td>

  <td style="margin: 10px">
<a href="https://paypal.me/iamtekson"><img src="https://www.paypalobjects.com/webstatic/mktg/Logo/pp-logo-200px.png" alt="PayPal Logo"
style="max-width:90%;" width="200" height="60">
 </td>
 </tr>
 </table>
</p> 
