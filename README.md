[![License](https://img.shields.io/github/license/bilelomrani1/s5p-tools.svg)](https://img.shields.io/github/license/bilelomrani1/s5p-tools.svg)

S5P-Tools
=====================================

A Python script to download and preprocess data from [Copernicus Open Access Hub](https://scihub.copernicus.eu). This implementation is based on `sentinelsat` [package](https://github.com/sentinelsat/sentinelsat) and the [API Hub Access](https://scihub.copernicus.eu/twiki/do/view/SciHubWebPortal/APIHubDescription) to query the database. The preprocessing is made with [HARP tools](https://cdn.rawgit.com/stcorp/harp/master/doc/html/harpconvert.html).

## Installation

We recommend using `conda` to create and manage a virtual environment when using this set of scripts. You can create a new virtual environment with all dependencies pre-installed using
```
conda create --override-channels -c conda-forge -c stcorp --file requirements.txt --name <envname>
# sentinelsat is not available through conda so we install it using pip
conda activate <envname>
pip install sentinelsat
```
Alternatively, to install all dependencies in an already-existing environment, use
```
conda install --override-channels -c conda-forge -c stcorp --file requirements.txt
pip install sentinelsat
```

> **Notes:** `conda` can take a while to resolve dependencies.

## Downloading and processing data

### Quick start

The script `s5p-request.py` is used to query Copernicus Hub, download and process the data. The syntax is the following:

```bash
python s5p-request.py <product-type>
```
where `<product-type>` is a Sentinel-5P product. TROPOMI Level 2 geophysical products are given in the table below.

| Product type | Parameter                                                         |
| ------------ | ----------------------------------------------------------------- |
| L2__O3____   | Ozone (O3) total column                                           |
| L2__NO2___   | Nitrogen Dioxide (NO2), tropospheric, stratospheric, slant column |
| L2__SO2___   | Sulfur Dioxide (SO2) total column                                 |
| L2__CO____   | Carbon Monoxide (CO) total column                                 |
| L2__CH4___   | Methane (CH4) total column                                        |
| L2__HCHO__   | Formaldehyde (HCHO) tropospheric, slant column                    |
| L2__AER_AI   | UV Aerosol Index                                                  |
| L2__CLOUD_   | Cloud fraction, albedo, top pressure                              |

By default, the script downloads all products corresponding to the specified product type for the last 24 hours. Custom date query can be specified with `--date`.

The resulting file is a `netCDF` file in the `processed` folder, binned by time, latitude, longitude, aligned on the same regular grid with resolution 0.01 x 0.01 arc degree by default. For parsing and plotting, we recommend the Python package `xarray`. An example of preliminary analysis can be found in the repo [s5p-analysis](https://github.com/bilelomrani1/s5p-analysis).

### Options

The script `python s5p-request.py` supports the following optional arguments:


| Product type    | Description                                   | Example                                                           |
| --------------- | --------------------------------------------- | ----------------------------------------------------------------- |
| `--date`        | Date used to perform a time interval search   | `python s5p-request.py L2__NO2___ --date 20200101 20200108`       |
| `--aoi`         | Path to the area of interest file (.geojson)  | `python s5p-request.py L2__NO2___ --aoi area_of_interest.geojson` |
| `--shp`         | Path to the shapefile for masking (.shp)      | `python s5p-request.py L2__NO2___ --shp shapefile.shp`            |
| `--unit`        | Unit conversion                               | `python s5p-request.py L2__NO2___ --unit Pmolec/cm2`              |
| `--qa`          | Quality assurance value threshold             | `python s5p-request.py L2__NO2___ --qa 50`                        |
| `--resolution`  | L3 grid spatial resolution in arc degrees     | `python s5p-request.py L2__NO2___ --resolution 0.1 0.1`           |
| `--num-threads` | Number of threads spawned for L2 download     | `python s5p-request.py L2__NO2___ --num-threads 2`                |
| `--num-workers` | Number of processes spawned for L3 conversion | `python s5p-request.py L2__NO2___ --num-workers 8`                |


#### Date

The `--date` option allows to perform a time interval search.

```bash
python s5p-request.py <product-type> --date <timestamp> <timestamp>
```
`<timestamp>` can be expressed in one of the following formats:
  - yyyyMMdd
  - yyyy-MM-ddThh:mm:ssZ
  - yyyy-MM-ddThh:mm:ss.SSSZ(ISO8601 format)
  - NOW
  - NOW-<n>MINUTE(S)
  - NOW-<n>HOUR(S)
  - NOW-<n>DAY(S)
  - NOW-<n>MONTH(S)

The first timestamp is included and the second is excluded.

#### Area of interest

The `--aoi` option allows to specify a custom geographical area with a `geojson` file.

```bash
python s5p-request.py <product-type> --aoi <geojson-file-url>
```
You can use [geoJSON.io](http://geojson.io) to generate a custom `.geojson` file for your area of interest.

#### Shapefile

The `--shp` option allows to mask the resulting dataset based on the geometry contained in a `.shp` shapefile.

```bash
python s5p-request.py <product-type> --shp <shapefile-file-url>
```
If the shapefile contains more than one geometry, the script considers the union of all geometries.

#### Unit conversion

By default, no unit conversion is performed (SI units). To perform unit conversion, use `--unit`.

```bash
python s5p-request.py <product-type> --unit <unit>
```

Unit conversion supports the following arguments for densities:
- `Pmolec/cm2`
- `molec/m2`
- `mol/m2` (default)

#### Quality value filtering

By default, the script filters all values whose quality value is below 75. This behavior can be adjusted with `--qa`.

```bash
python s5p-request.py <product-type> --qa <int>
```

#### Spatial resolution

By default, the script uses a 0.01x0.01 arc degrees resolution grid during the L3 conversion. This resolution can be adjusted with `--resolution`.

```bash
python s5p-request.py <product-type> --resolution <float> <float>
```

#### Number of threads

By default, the script spawns 4 threads when downloading from SentinelHub. This number can be adjusted with `--num-threads`.

```
python s5p-request.py <product-type> --num-threads <int>
```

#### Number of workers

By default, the script spawns a number of processes equals to the number of virtual cores the when performing L3 conversion. This number can be adjusted with `--num-workers`.

```
python s5p-request.py <product-type> --num-workers <int>
```

## Acknowledgements

The authors are grateful to the Luxembourg Institute of Socio-Economic Research (LISER) for funding this project. The authors would also like to acknowledge the European Spatial Agency for providing the API for the Sentinel 5P Hub. The content is solely the responsibility of the authors and does not necessarily represent the official views of the LISER.

If you use this code, please consider citing the following paper.

```
@article{OMRANI2020105089,
title = "Spatio-temporal data on the air pollutant nitrogen dioxide derived from Sentinel satellite for France",
journal = "Data in Brief",
volume = "28",
pages = "105089",
year = "2020",
issn = "2352-3409",
doi = "https://doi.org/10.1016/j.dib.2019.105089",
url = "http://www.sciencedirect.com/science/article/pii/S2352340919314453",
author = "Hichem Omrani and Bilel Omrani and Benoit Parmentier and Marco Helbich",
keywords = "Air pollution, Remote sensing, Monitoring",
abstract = "Monitoring of air pollution is an important task in public health. Availability of data is often hindered by the paucity of the ground monitoring station network. We present here a new spatio-temporal dataset collected and processed from the Sentinel 5P remote sensing platform. As an example application, we applied the full workflow to process measurements of nitrogen dioxide (NO2) collected over the territory of mainland France from May 2018 to June 2019. The data stack generated is daily measurements at a 4 × 7 km spatial resolution. The supplementary Python code package used to collect and process the data is made publicly available. The dataset provided in this article is of value for policy-makers and health assessment."
}
```
