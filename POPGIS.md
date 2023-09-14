POPGIS - Developer Guides
====================
***Improving Air Pollution Monitoring and Management of Vietnam with satellite PM2.5 observation***
***Laser Pulse Project***

# Overview

This document provides technical details related to implementations of POPGIS. POPGIS is a software built for 
showing air quality (PM2.5 and AQI maps) over Vietnam territories. It is supported by project “Improving Air Pollution Monitoring and Management of Vietnam with satellite PM2.5 observation” which is underscope of Laser Pulse Project. 

The “Improving Air Pollution Monitoring and Management of Vietnam with satellite PM2.5 observation” project aims to construct a complete dataset of PM2.5 distribution of the Vietnam region, as well as to develop translation products to provide comprehensive information on the PM2.5 nationwide to the public (including datasets, reports, videos, WebGIS), and disseminating these products to end users. These products will hopefully assist the public and government to take action to reduce the effects of air pollution on the environment and human health. Remote sensing technologies using satellites are implemented in this project to overcome the drawbacks of on-ground monitoring methods, along with machine learning techniques to create air pollution maps of Vietnam. We are collaborating with multiple stakeholders to create the suitable translation products for the public.

Particularly, the following information will be provided:

- POPGIS technical details (architecture, how it works, etc.)
- POPGIS deployment guides
- POPGIS development guides

# POPGIS Architecture
![](https://lh3.googleusercontent.com/drive-viewer/AFGJ81oU26w1FT3k4wf-tqgh3zCANa174y5IJGR3xwhK4y4nCaZIYRksyYeYQERk7qgCRpfWLVenimIdzVRbHDpWUTrCqTTs8g=w3360-h1896)

*Figure 1: POPGIS System Architecture*

The POPGIS system architecture is shown in figure 1. It comprises of 02 subsystems: (i) Processing Pipelines and (ii) POPGIS Web Application. 

- POPGIS Processing piplines: This subsystem take responsibility for retrieving data from *external sources* and processing the data to produce outputs (air quality maps). This subsystem also contains multiple supporting utilities like pipeine management tools.
- POPGIS Web Application: presents all result data to user via Web-based interfaces 

External data sources that POPGIS system relied on are listed in the following tables:

| Item | Short name       | Description                                                | Provided by                    |
|------|------------------|------------------------------------------------------------|--------------------------------|
|   1  | MOD              | MODIS Terran                                               | NASA                           |
|   2  | MYD              | MODIS Aqua                                                 | NASA                           |
|   3  | VIIRS            | Common utils                                               | NASA                           |
|   4  | NDVI             | Application entry                                          | NASA                           |
|   5  | RDA              | NCAR Research Data Archive (RDA)                           | NCAR                           |
|   6  | Air Station data | Air station data in Vietnam (Haoi EPA, USE-USC, UWYO, VEA) | Vietnam government, US embassy |
|   7  | OSM | Open Street Map | Open Street Map Project |
*Table 1: External data sources*

## Processing Pipelines
The Processing Pipelines Subsystem contains multiple *pipelines*. Each pipeline is set of related processing *stages* (which are processing mini applications or scripts). In a *pipeline* outputs of a stage could be inputs for another stages. A *dependency graph* is setup within the POPGIS source code that defines such dependency between those stages in the subsystem. Details of *dependency graph* are shown in [Job Queue and Scheduling](#Job_Queues_and_Scheduling) 

The Processing Pipeline Subsystem is designed to run on a cluster of servers (either physical or virtual). Servers of the cluster are called *Worker nodes*. Pipelines of the subsystem should be deployed on worker nodes and worker nodes will take responsibility to run the deployed pipeline regularly. There are 5 pipelines in the subsystem. 

- WRFChem Pipeline : Process WRFChem model (See [WRFChem Pipeline](#WRFChem_Pipeline_56))
- MEMNRT Pipeline: Process Air quality maps (See [MEMNRT Pipeline](#MemNRT_Pipeline_57))
- MEM Pipeline: Process others intermedia map for MapNRT pipeline (See [MEM Pipeline](#MEM_Pipeline_58))
- Station pipeline: Process Air quality from observation stations (See [Station Data Pipeline](#Station_Data_Pipeline_59))
- Publish pipeline: pushlishing data to POPGIS Application. (See [Publish Pipeline](#Publish_Pipeline_60))

*Worker nodes* share data between one another using network shared folders (e.g.: Network File System (NFS) or Server Message Block (SMB)). Pipelines that required shared folder includes *WRFChem Pipeline*, *MEMNRT Pipeline* and *MEM Pipeline*

THe Processing Pipeline Subsystem also contains: 

- A management application, *Pipeline Tracker*, that monitor and control execution of pipeline
- and a job queue (Redis) to coordinates worker nodes.

Detailed architecture of Processing Pipeline Subsystem are shown in figure 2.

![](https://lh3.googleusercontent.com/fife/APg5EOamKQUAT2psHeWgizfZQM7ca4EpcMIfKCLJ9cFhG-uAJyBx8hnbsfhqSgdBhBjQ-YizVP8TRbSu1XmH_6yXTZbLoSFhwx5udEy-TIOi3nK16r0HwlVXLLskV3jMFv0Ap8zAVQJsMRku8N-VrvfLQDbYMOfkilu_VnToruUdQhJluE3OhFHanekIahPjxsX1bTBN-bg0qp2jpQg97yQcTt7ajJ3UDLvExShpDaQptNlFZpA-VO24Sz2MX8PoseNkFdDgZdFBe8rdJcuvWZ6otT6DRE0RuIzf2GVguYQO6E7DfgoG-nspJYL6ck_jBEKuiMy6GxDE29KKxXThPxWyew4rJNbN93DQ5ozMVvO5dnoAklM6xvEcCYrRfGMh7qGBEzYFzc3pvsv5XmzIhknmejNAcrZvr6mVSqi34qcVjfQ8ipqva715a51aRzP0W0uUpW7SHTO68HnE8S6AZ5uoSoVultMCZPRZRl9Dbbrg4cT5ePsVKgtKd7KfTYbofmEm4HG8XV7nZKUlaj-Sjg-stUhiJHZ7_D2twXDtD6PKf39lRIZQhetnQ3ztix_fszafhi1JiI1s8ZVM1Lkn-ERZ-6io_4UMQOU4YwkNfoxiQKdbPJIeZ-Le9I41lAghlxyEexrinZkTCjmKDUKtQeOE3u_5jB-SIOmYFFOHR1HSbjySuGm2T9ZX1-Zdm3voN-PGaEBs5_OI-W9MNfA1nQUKXG6SagkEgSUGzAUr4W1pcdYCdnnDPuAn-tqNasTQ22DvcvTd6wwLduUgAtGf1g50tG5neJ63sayV8eouwe01bZss0DZaSt6DS8j34eYcWrANj36A742oreiE2tH_qgmD2j5axXAgbJd9VBK3iP9jSSwiHu7ATUvLO2lMRsrNhEpneQOtDKfPEiaHhPt5NyHw1qnUiuMO0CPUNURU6WVcrGKQmM-EghQUTQ67LOau7GP7oxZYtow5YTo2QvdAqrH3civ2UrVJecKkbgwQI-z1Enor46gPivQu_n_odnd62ElqInYr19TRQuFdR08wpK0r0IZuLv8S31iohBi6rSXuKr0M4D0c4aJxebTjVu1a6ctu6JqxSIyak-M0IzA8v9f14aT1QuinGnWY0BA1ivjQwnRGHJ4emmK8Gt7wUxVPI1UPf2D_HFDwTnVNDtzBoP1Jz-q4dqqeszaC1AQrrZ1O--NpkdOzm61SKYUGfHl0ksLBOshkxU3tuzcm5cOxS4tFOO1-Gr8Nkusn5A_KXgoCiir-ak50JQDh4Ab1QvWRYn3-IFDemM7Z1miSQyGj_R_1SVOHc4BXwn1GGa5m2la0tTrNNNylYctSeQELnzg6VYOVs63wylkRKHDDiFCRcVYBO9CDuVuD0RU2jh5zBdIZn9NojMsS4A8CY0evEFnC0_gNGdifpSHN_exEmqh3OQ0hlkYHMcFG4w3TWIs4ZfeU7bD4-R-qA0mznTn2gH97aFxK8OG_B3O-ukKppgAj5ueGKUln3lZVv--0EDWzoLgz19_GEZ3GcCi1XjYS4J5VmDFyDig=w2508-h1782)

*Figure 2: Processing Piplines Subsystem with 2 worker nodes*

On figure 2, the 5 pipelines are deployed on 2 worker nodes. Note that WRFChem, MemNRT, MEM are deployed on both nodes and they requires a NFS mount and an object store (Minio or S3). The two other important components are a job queue and a Pipeline Tracker application that allows administrator to control how the whole pipeline runs.

### WRFChem Pipeline
WRFChem Pipeline is a pipeline that takes responsibility for building meteorological data. This pipeline runs daily. The ultimate results of the pipeline is used by stages in MapNRT pipeline. List of stages in WRFChem models are listed in the following table (table 2)

| SeqNo | Stage             | Description                                        |
|-------|-------------------|----------------------------------------------------|
|   1   | GetFNL            | Retrieve data (FNL - Final Analysis) from NCAR RDA |
|   2   | runWPS            | Run WRF preprocessing                              |
|   3   | runREAL           | Run WRF real-data simulation                       |
|   4   | runEMISS          | WRF preprocessing for emission model               |
|   5   | runWRF            | Run WRF prediction model                           |
|   6   | WRFpostprocessing | WRF post processing                                |
|   7   | UploadFile        | Save result files to object storage (Minio)        |
|   8   | UploadToDB        | Save result to Database                            |
*Table 2: Stages of WRFChem pipeline* 

This pipeline uses data from NCAR RDA (See [table 1](#Table_1_External_data_sources_34)).
This pipeline requires access to NFS/SMB share folder and object storage. For details on how WRFChem pipeline is deployed, see [Pipeline Worker Nodes](#Pipeline_Worker_Nodes)

### MemNRT Pipeline

MemNRT Pipeline is the pipeline that takes responsibility to build the final output, which is PM2.5 maps. This pipeline use:

- MODIS Data: MOD, MYD, VIIRS, NDVI 
- WRFChem Pipeline output 

This pipeline run daily. Output of this pipeline will be publish to POPGIS Application by Publish Pipeline. Stages of this pipeline is listed in the following table

| SeqNo | Component         | Description                            |
|-------|-------------------|----------------------------------------|
|   1   | downloadData      | Download MOD, MYD, and VIIRS datasets  |
|   2   | ImgProcess        | Preprocess of MOD, MYD, VIIRS images   |
|   3   | WRRChemProcessing | Preprocess of WRFChem output images    |
|   4   | PM25predict       | Build multimodal PM2.5 map             |
|   5   | UploadFile        | Save results to Object storage (minio) |
|   6   | UploadToDB        | Save results to Database               |
*Table 3: Stages of MemNRT pipeline*

### MEM Pipeline

MEM Pipeline downloads and processes NDVI data (Normalized Difference Vegetation Index). This will be used in MemNRT Pipeline (PM25predict stage). This runs daily and contains only two stages: `downloadNDVI` and `NDVIPreprocess`.

### Station Data Pipeline

Station Data Pipeline downloads air station data from various organizations. It runs daily. It contains 1 stage: `stationData`

### Publish Pipeline

Publish Pipeline will be triggered by MemNRT. It contains a single stage `memNRTAdd`

### Job Queues and Scheduling
Pipelines worker nodes are coordinated and scheduled using a job queue. When a worker node is started, it will wait on the job queue for dispatched jobs. There can be multiple worker nodes waiting on a single queue. To kick off a stage of a pipeline, the thing needs to be done is inserting a new job into the job queue. 

Stages of pipelines are dependent on each other on a daily basis. For example, on a day `d`, output of stage 1 is input for stage 2. Therefore, we can construct a directed acyclic graph (or DAG) among all stages using such a relationship. The DAG helps the system to automatically fire stages. For example, if all upstream stages of a specific stage are successfully finished, the system can trigger the stage immediately. Thus, to run all pipelines, only stages that have no dependency, or `seed stages` should be triggered. All other stages will be invoked by the system. Dependency graph between stages are shown in figure 3

![](https://lh3.googleusercontent.com/drive-viewer/AFGJ81rxVWG3rWcMs3A9pVBefy-E5IpB5Hbv1Nj6sXWs3iuJzcxGtgcTg4SG89n44a49dETgzFywisz4lp_m8Ftm3PFE6wG9lA=w3360-h1896)

*Figure 3: Dependency graph between stages* 

There are cronjobs setup to schedule `seed stage` regularly. By that way, all pipelines of the system will run accordingly. Also, if an execution of a job fails, it can be rerun manually via Pipeline Tracker application . Detailed ìnformation about those cronjobs can be found in [Cronjobs Installation](#Cronjobs_Installation). For installation and administration of a worker node, see [Pipeline Worker Nodes](#Pipeline_Worker_Nodes). For manually control pipeline executions via Pipeline Tracker, see [Pipeline Tracker Application](#Pipeline_Tracker_Application).  

## POPGIS Web Applications

POPGIS Web Application is built using Microsoft .NET framework and OSGeo GeoServer. Both application servers are backed by PostgreSQL Database. Detailed structure of POPGIS Web Application are shown in the following figure (Figure 4). Its components are summarized in table 3

![](https://lh3.googleusercontent.com/drive-viewer/AFGJ81o9UAauvTj4id5TE4mrNtHrWVJFXs7moKcxiV3t5tWtcFPSbVDxeePBlO6GowsCZDnFl66ZdgiIuGqhpY5OlePTvbBOZw=w3360-h1188)

*Figure 4: POPGIS Web Application*

| SeqNo | Component  | Description                                 |
|-------|------------|---------------------------------------------|
|   1   | APOM User  | .NET project. View layer of the application |
|   2   | APOM API   | .NET project API layer                      |
|   3   | GeoServer  | WebGIS and map layer                        |
|   4   | PostgreSQL | Application Database                        |
*Table 4: POPGIS Web Application components*

# POPGIS System Deployment

## Organizing and Planing
POPGIS system contains various components and should be deployed on multiple servers. For the ease of explaination, we explain how to deploy POPGIS on a sample scenario with 4 physical servers, called S1, S2, S3, and S4. The 4 servers are interconnected (e.g. in the same LAN). At least one of the servers should be accessible from public Internet for providing service.

Placements of POPGIS components on the 4 servers are shown in the following table (Table 5)

| **SeqNo** | **Component**             | **Subsystem**          | **Deploy at** |
|-----------|---------------------------|------------------------|---------------|
|     1     | WRFChem pipeline          | Processing Pipelines   |    S1 & S2    |
|     2     | MemNRT pipeline           | Processing Pipelines   |    S1 & S2    |
|     3     | MEM pipeline              | Processing Pipelines   |    S1 & S2    |
|     4     | Publish pipeline          | Processing Pipelines   |    S1 & S2    |
|     5     | Station Data pipeline     | Processing Pipelines   |       S3      |
|     6     | Redis Server              | Processing Pipelines   |       S3      |
|     7     | Minio Server              | Processing Pipelines   |       S3      |
|     8     | NFS server                | Processing pipelines   |       S3      |
|     9     | Pipeline Tracker Backend  | Processing Pipelines   |       S4      |
|     10    | Pipeline Tracker Frontend | Processing pipelines   |       S4      |
|     11    | Nginx                     | Supporting modules     |       S3      |
|     12    | APOM User & APOM API      | POPGIS Web Application |       S4      |
|     13    | GeoServer                 | POPGIS Web Application |       S3      |
|     14    | PostgreSQL server         | POPGIS Web Application |       S3      |
|     15    | Community site (Hugo)     | Supporting modules     |       S3      |
*Table 5: Sample deployment scenario*

Source code repositories (Gitlab) used for deploying POPGIS are summarised in Table 6

| **SeqNo** | **Source repo**           | **Source URL**                                                 | **Notes**                 |
|-----------|---------------------------|----------------------------------------------------------------|---------------------------|
|     1     | apom_pipeline             | https://gitlab.com/hapv241/apom_pipeline.git                 | All pipelines of POPGIS   |
|     2     | pipeline-tracker          | https://gitlab.com/tung.hoang2/pipeline-tracker.git          | Pipeline Tracker Backend  |
|     3     | pipeline-tracker-frontend | https://gitlab.com/tung.hoang2/pipeline-tracker-frontend.git | Pipeline Tracker Frontend |
|     4     | apom_app                  | https://gitlab.com/hapv241/apom_app.git                      |    APOM User & APOM API   |
*Table 6: POPGIS source code repositories*

Remaining components are open source softwares that we reused for the project. Their names and versions are summarised in table 7. They are versions/distributions that we successfully used for our POPGIS system.

| **SeqNo** | **Source repo** | **Version/Notes**        |
|-----------|-----------------|--------------------------|
|     1     | Redis           |           5.0.7          |
|     2     | Nginx           |       1.18.0/Ubuntu      |
|     3     | GeoServer       |          2.20.0          |
|     4     | NFS             | nfs-utils-1.3.0 (CentOS) |
|     5     | Minio           |       RELEASE.2021       |
|     6     | PostgreSQL      |        13.5 Ubuntu       |
|     7     | Hugo            |     v0.68.3/extended     |
*Table 7: third-parties and their versions*

## Pipeline Worker Nodes

### Installation Worker Node for WRFChem, MemNRT, MEM pipelines
Linux environment is required for running WRFCHem, MemNRT and MEM pipelines. The recommended Linux distribution is CentOS 7. Roadmap for installation of execution environment of those pipeline is described belows:

- Install PGI Compiler 
- Build dependent libraries from source using PGI compiler (the required libraries are OpenMPI, zlib, libpng, Jasper, HDF5, NetCDF, WRF, WPS, etc)
- Mount NFS drive for share data
- Install and configure Redis
- Install and configure Minio
- Prepare Python environment
- Deploy Pipelines' source code 
- Configure each pipelines

All the above steps should be done on worker nodes that run WRFChem, MemNRT, MEM pipelines. For the sample scenario, they should be done on S1 and S2.

#### Install PGI Compiler

We used PGI Compiler included in NVDIA's HPC SDK version 20.11. Other version of HPC SDK would also work. However we have not tested all HPC SDK version except for version 20.11. Ones can refer to this link (https://developer.nvidia.com/nvidia-hpc-sdk-2011-downloads) for installation of the HPC SDK version. In our case, we can install it with the following commands (assuming root account privileges)

```
# yum-config-manager --add-repo https://developer.download.nvidia.com/hpc-sdk/rhel/nvhpc.repo
# yum install --nogpgcheck nvhpc-20.11
```

After executing the above commands, the software will be installed at `/opt/nvidia`. PGI compilers (c/c++, fortran, ...) are located in `/opt/nvidia/hpc_sdk/Linux_x86_64/20.11/compilers/bin/`. This should be added into `PATH` environment.

```
# export PATH=/opt/nvidia/hpc_sdk/Linux_x86_64/20.11/compilers/bin/:$PATH
```

#### Building dependent libraries

The following libraries should be rebuilt from their source code, using PGI compiler: 

- OpenMPI (https://www.open-mpi.org/): tested version 2.1.6
- zlib (https://www.zlib.net/): tested version 1.2.11
- libpng (http://www.libpng.org/pub/png/libpng.html): tested version 1.2.50
- jasper (https://www.ece.uvic.ca/~frodo/jasper/): tested version 1.900.1
- HDF5 (https://support.hdfgroup.org/ftp/HDF5/releases/): tested version 1.14.0
- NetCDF (https://www.unidata.ucar.edu/downloads/netcdf/index.jsp): tested version 4.6.1
- NetCDF Fortran (https://www.unidata.ucar.edu/downloads/netcdf/index.jsp): tested version 4.4.4
- WRFV3, WRFV3-Chem (https://github.com/wrf-model/WRF/releases/tag/V3.8.1): tested version 3.8.1
- WPS (https://github.com/wrf-model/WPS/releases/tag/RELEASE-3-8-1): tested version 3.8.1
- WRF-CHEM Processor (https://www2.acom.ucar.edu/wrf-chem/wrf-chem-tools-community): megan, ANTHRO, wes-coldens. We have prepared those source code and appropriate input files that are ready for build. 
- UPPV3-CHEM (https://dtcenter.org/community-code/unified-post-processor-upp): tested version 3.0 

How to build those libraries can be found in their source code or in their websites. For ease of recontruction the environment, we includes the steps for building those libraries belows as reference. Notes that it may need adjustments depending on your sittuations. In our situation, we used a common directory `$PREFIX` to store install binaries

```
# export PREFIX=/opt/pgi_lib
```

Also, make sure the following environment variables are set

```
# export PATH=$PREFIX/bin:$PATH
# export LD_LIBRARY_PATH=$PREFIX/lib:$LD_LIBRARY_PATH
```

**Building OpenMPI**

Expand source code

```
# tar -xvzf openmpi-2.1.6.tar.gz
# cd openmpi-2.1.6
```

Configure and compile and install 

```
# ./configure --prefix=$PREFIX --disable-shared --enable-static
# make
# make install
```

**Building zlib**

Expand source code

```
# tar -zxvf zlib-1.2.11.tar.gz
# cd zlib-1.2.11
```

Configure and compile and install

```
# configure --prefix=$PREFIX
# make
# make check
# make install
```

**Building libpng**

Expand source code

```
# tar -zxvf libpng-1.2.50.tar.gz
# cd libpng-1.2.50
```

Configure, compile, and install

```
# configure --prefix=$PREFIX 
# make 
# make check
# make install
```

***Building Jasper***

Expand source code

```
# tar -zxvf jasper-1.900.1.tar.gz
# cd jasper-1.900.1
```

Configure, compile and install

```
# configure --prefix=$PREFIX
# make
# make check
# make install
```

***Building HDF5***

```
# tar -xvzf hdf5-1.14.0.tar.gz
# cd hdf5-1.14.0
# ./configure --prefix=$PREFIX --with-zlib=$PREFIX --enable-shared --enable-fortran --enable-static
# make
# make check
# make install
```

***Building NetCDF for C***

```
# tar -xvzf netcdf-c-4.6.1.tar.gz
# cd netcdf-c-4.6.1
# ./configure --prefix=$PREFIX --disable-netcdf-4 --enable-shared --enable-static
# make
# make check
# make install
```

***Building NetCDF for Fortran***

```
# tar -xvzf netcdf-fortran-4.4.4.tar.gz
# cd netcdf-fortran-4.4.4
# ./configure --prefix=$PREFIX --disable-netcdf-4 --enable-shared --enable-static
# make
# make check
# make install
```

***Building WRF***

Add the following environment variable

```
# export WRF_CHEM=1
# export WRFIO_NCD_LARGE_FILE_SUPPORT=1
```

Expand WRF source and move into WRF source folder. Then issue the following command

```
# ./configure
```

Select option 2 (Linux x86_64 i486 i586 i686, PGI compiler with gcc  (smpar)), and then option 1 (Basic compile for nesting). 

Then perform compilation

```
# ./compile em_real
```

Successfull compilation will result in the following files: `run/real.exe`, `run/wrf.exe`, `run/tc.exe`, `ndown.exe`.

***Building WPS***

Expand WPS source and move into the source folder. Perform configuration 

```
# ./configure
```

Select option 5. Then perform compilation

```
# ./compile
```

A successful compilation results in the following files `geogrid.exe`, `metgrid.exe`, and `ungrib.exe`

***Building WRF-Chem Processors***

Expand PRE_WRF.tar.gz and move into the new folder

```
# tar -xvzf PRE_WRF.tgz
# cd PRE_WRF
```

Build megan processor

```
# cd megan
# make
cd ..
```

Build ANTHRO processor

```
# cd ANTHRO/src
# make
cd ../..
```

Build wes-coldens

```
# cd wes-coldens
# make
```

Move the three processors to `run` directory inside WRFv3 location

```
# mv megan <WRFv3 location>/run
# mv ANTHRO <WRFv3 location>/run
# mv wes-coldens <WRFv3 location>/run
```

***Build UPPV3_Chem***

Extract tar ball and move into the directory.

```
# tar -xvzf UPPV3_Chem.tgz
# cd UPPV3_Chem
```

Adjust build parameter in `configure.upp` as follows

```
WRF_DIR         =    <Location of WRFv3 installation>
WRF_LIB2        =    
NETCDFPATH      =    <Location of $PREFIX>
NETCDFLIBS      =    -lnetcdff -lnetcdf
...
GRIB2SUPT_LIB   =    -L<Location to $PREFIX> -lpng -lz -ljasper 
GRIB2SUPT_INC   =    -I<Location to $PREFIX>

BINDIR          =    <Location to UPPV3_Chem>/bin
INCMOD          =    <Location to UPPV3_Chem>/include
LIBDIR          =    <Location to UPPV3_Chem>/lib
```

Perform configure and build

```
# ./configure
# ./compile
```

#### Mount NFS drive for sharing data

All worker nodes that execute WRFChem, MemNRT, MEM pipelines needs to store their results into a share folder, called `$PIPELINE_OUTPUT_ROOT`. Specifically, $PIPELINE_OUTPUT_ROOT should be prepared and exported on S3. S1 and S2 will mount the share to their local directory tree. We suggest a layout for $PIPELINE_OUTPUT_ROOT as follows:

```
$PIPELINE_OUTPUT_ROOT
  ├ wrfchem-pipeline
  │   ├ error
  |   ├ input
  |   ├ log
  |   └ output
  ├ memNRT-pipeline
  |   ├ error
  |   ├ input
  |   ├ log
  |   └ output
  ├ mem-pipeline
  |   ├ error
  |   ├ input
  |   ├ log
  |   └ output
  ...
```

Install NFS server on S3 

```
# apt update
# apt install nfs-kernel-server
```

Prepare the share (e.g. /opt/pipeline_data) and export it by adding an entry to `/etc/exports`

```
...
/opt/pipeline_data   192.168.0.0/24(rw,sync,no_root_squash,no_all_squash)
...
```

You should adjust `192.168.0.0/24` to the subnet of your worker nodes. 

From S1 and S2, perform mounting to the share by adding entry to `/etc/fstab`

```
...
<S3 IP address>:/opt/pipeline_data        <Location of $PIPELINE_OUTPUT_ROOT>     nfs     defaults        0 0
...
```

***Note:*** the share folder are recommended to be sufficiently large (several TB) for pipeline output.

Install nfs client at S1 and S2 ...

```
# yum install nfs-utils # for CentOS
# apt install nfs-utils # for Ubuntu
```

..., then mount to the share
```
# mount <Location of $PIPELINE_OUTPUT_ROOT>
```

#### Install and config Redis server

Redis is required for coordinating pipelines. According to the sample scenario, Redis server is deployed on S3.

Install Redis on S3

```
# apt install redis-server
```

Configure the redis server to listen on all interfaces with password authentication. Change `/etc/redis/redis.conf` to make sure that the file has following lines

```
bind 0.0.0.0
protected-mode yes
requirepass <redis password>
```

Make redis autostart

```
# systemctl enable redis-server
# systemctl start redis-server
```

#### Install and configure Minio

Minio is also required for pipelines. We install and configure minio on S3 with the following commands (Assuming Ubuntu system)

```
# wget https://dl.min.io/server/minio/release/linux-amd64/minio_20220523184511.0.0_amd64.deb
# dpkg -i minio_20220523184511.0.0_amd64.deb
# groupadd -r minio-user
# useradd -M -r -g minio-user minio-user/
# chown minio-user:minio-user <Storage volume/dir for minio>
```

***Note:*** Storate volume / directory for minio should have large capacity (several TB) for storing data.

Add the following lines to `/etc/default/minio`

```
MINIO_VOLUMES="<Storage volume/dir for minio>"
MINIO_OPTS="--certs-dir /etc/minio --console-address :9001"
MINIO_ROOT_USER=<$MINIO_USERr>
MINIO_ROOT_PASSWORD=<$MINIO_PASS>
```

Start minio and make it autostart

```
# systemctl start minio
# systemctl enable minio
```

#### Prepare Python Environment
The repository for POPGIS's pipelines is `apom_pipeline` (see table 6). Ones can get fetch the source code to local machine using `git clone` command. Python environment is needed for running `apom_pipeline`. The recommended Python distribution is Anaconda 3 (https://www.anaconda.com/download) 

Additional required Python packages are listed in the following tables (table 9). They need to be installed in to the running conda environment using either `conda` or `pip` utility. You can also consult `requirement.txt` in apom_pipeline source code to install the dependencies.

| **SeqNo** | **Python moduls**   | **Version/Notes** |
|-----------|---------------------|:-----------------:|
|     1     | beautifulsoup4      |       4.11.1      |
|     2     | GDAL                |       3.4.1       |
|     3     | Jinja2              |       3.1.2       |
|     4     | cdsapi              |       0.5.1       |
|     5     | numba               |       0.56.4      |
|     6     | numexpr             |       2.8.4       |
|     7     | numpy               |       1.23.5      |
|     8     | minio               |       7.1.0       |
|     9     | pandas              |       1.5.2       |
|     10    | psycopg2            |       2.9.3       |
|     11    | pyhdf               |       0.10.2      |
|     12    | redis               |       4.4.2       |
|     13    | requests            |       2.28.1      |
|     14    | rq                  |       1.11.1      |
|     15    | scikit-learn        |       1.1.3       |
|     16    | scipy               |       1.9.3       |
|     17    | openpyxl            |       3.0.10      |

*Table 9: Major python packages (dependencies) for WRFChem, MemNRT and MEM pipelines*

#### Pipeline Configurations

The table belows summarized important locations and parameters that is used to pipeline configuration steps

| **SeqNo** | **Directory/File**     |                  **Notes**                 |
|-----------|------------------------|:------------------------------------------:|
|     1     | $PREFIX                | Where all compiled libraries are installed |
|     2     | $WRFv3                 | Where WRF Chem models are built            |
|     3     | $WRFv3/run             | Where WRF chem will run                    |
|     4     | $WRFv3/run/ANTHRO      | ANTHRO Processor                           |
|     5     | $WRFv3/run/megan       | Bio emis processor                         |
|     6     | $WRFv3/run/wes-coldens | wes-coldens processors                     |
|     7     | $WPSv3                 | Where WPS is installed and run             |
|     8     | $UPPV3_Chem            | Where UPPV3 chem is installed              |
|     9     | $PIPELINE_OUTPUT_ROOT  | Where the output of pipeline stored        |
|     10    | $PIPELINE_SRC_ROOT     | Root of pipeline source code               |
|     11    | $REDIS_URL             | Redis URL                                  |
|     12    | $MINIO_URL             | Minio location                             |
|     13    | $MINIO_USER            | Minio account                              |
|     14    | $MINIO_PASS            | Minio password                             |
|     15    | $DB_HOST               | POPGIS Web app database host               |
|     16    | $DB_PORT               | POPGIS Web app database port               |
|     17    | $DB_NAME               | POPGIS Web app database name               |
|     18    | $DB_USER               | POPGIS Web app database user               |
|     16    | $DB_PASS               | POPGIS Web app database pass               |

*Table 10: Summary of important locations for pipeline execution environment*

***Configure WRFChem pipeline***

Modify the following variables in `$PIPELINE_SRC_ROOT/<pipeline_name>/config.py`

```
PIPELINE="<Pipeline name>"
DATA_BASE=fr'<Location of $PIPELINE_OUTPUT_ROOT>/{PIPELINE}'
SCRIPT_BASE=fr'<Location of $PIPELINE_SRC_ROOT>/{PIPELINE}'
```

Where <pipeline name> is `wrfchem_pipeline`, `mapNRT_pipeline`, and `map_pipeline` for WRFChem, MemNRT, MEM pipeline respectively.

Also, update Minio, Database parameters on those files.

### Installation Worker Node for other pipeline pipelines

Installation on Worker nodes that run other pipelines (Station data and publish pipeline) is similar to the procedure above. However, the following items are not necessary:

- PGI Compiler
- Building dependent libraries from sources
- Mount NFS

## Pipeline Tracker Application

Pipeline Tracker application is planed to deploy on server S3.

### Deploying Pipeline Tracker backend

Checkout `pipeline-tracker` repository (see table 6) to `$PIPELINE_TRACKER_BACKEND` directory using `git clone` command. Move into $PIPELINE_TRACKER_BACKEND directory and install python dependencies. (You'd better create a new isolated virtual environment for that)

```
# virtualenv .appenv
# . .appenv/bin/activate
# pip install -r requirement.txt
```

Now the Pipeline Tracker backend application can be started by running `start.sh` script

```
# ./start.sh
```

By default, Pipeline Tracker backend listens on port 8000. Pipeline Tracker backend configuration can be fine tuned by modifying the file `gconfig.py` and `start.sh`.

### Deploying Pipeline Tracker frontend

Checkout `pipeline-tracker` repository (see table 6) to `$PIPELINE_TRACKER_FRONTEND` directory using `git clone` command. Move into $PIPELINE_TRACKER_FRONTEND directory and install node dependencies. 

```
# npm install
# npm run build
```

After the above commands, `dist` directory is created holding all compiled frontend code and ready to work.

### Configure Nginx proxy for Pipeline Tracker

The following nginx configuration snippet can be used for Pipeline Tracker. This is our current working configuration. You can modify it to make it better suited for your sittuation.

```
server {
  root /var/www/html;
  # Add index.php to the list if you are using PHP
  index index.html index.htm index.nginx-debian.html;

  server_name pptracker.duckdns.org;

  location /api {
    rewrite /api/(.*) /$1  break;
    proxy_pass http://pptracker.duckdns.org:8000;
  }
  location / {
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;
    root /home/apom/pipeline-tracker-frontend/dist;
    try_files $uri $uri/ =404;
  }

  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/popgissvc.duckdns.org/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/popgissvc.duckdns.org/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    if ($host = pptracker.duckdns.org) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
    listen 80;
    server_name pptracker.duckdns.org;
    return 404; # managed by Certbot
}
```

The important things in the above configuration are two `location` blocks. The first block proxies requests that start with `/api` to Pipeline Tracker backend. The second block serves frontend code to clients with HTTP's basic authentication. One can remove two first lines in the second `location` block if he/she just wants to see how Pipeline Tracker works.

Once Pipeline Tracker application is started, we can view its user interface by visiting to its domain name (see figure 5). Through Pipeline Tracker, we can observe execution of stages in pipelines via their colors (Gray means "not running", Green means "success", Maroon means "Failure"). We can also manually invoke a stage by clicking on it to activate a dropdown menu and then selecting a desired action from the menu.

![](https://lh3.googleusercontent.com/drive-viewer/AFGJ81rY_vZjEFeiiseStpS29bwGLkTbFc7Opui0NU4ZarMKDYET-OfkFnwsU8Q7I6jxLm0Nx0-Z_xqSxEj0X6J7DQc1o0BprQ=w3360-h1896)

*Figure 5: User interface of Pipeline Tracker. (1) list of workers; (2) All stages of pipelines for a day; and (3) a pipeline stage*

## Running a Worker

To start a worker, from a worker node terminal, issue the following command

```
# rq worker [queue_name] --url redis://*:<$REDIS_PASS>@<$REDIS_URL> --with-scheduler
```

You should replace his/her Redis password and Redis url into the above command. Also the third word (queue name) is optional. If you want to start a worker for WRFChem, MemNRT, MEM pipeline, you should omit queue name (or use "default" for the queue name). If you want to start worker for other pipelines, `secondary` should be used for the queue name.

If a worker is started, it will show on the worker list in Pipeline Tracker's user interface

## Cronjobs Installation

For automatically triggering pipelines every day, three cronjobs for `wrfchem_pipeline.GetFNL`, `mapNRT_pipeline.downloadData`, and `map_pipeline.downloadNDVI` should be installed. Those are seed stages. Other stages will be invoked automatically using their dependencies. Belows are sample cronjob setting for your reference


```
# Cronjobs
35 1,2,3 * * * /bin/bash /opt/cronjobs/wrfchem.sh
5 11,12 * * * /bin/bash /opt/cronjobs/ndvi.sh
5 17,18 * * * /bin/bash /opt/cronjobs/mapNRT.sh
```

Content of `wrfchem.sh`

```
#!/bin/bash
today=$(date +%Y%m%d)
REDIS_HOST=<$REDIS_HOST> REDIS_PASS=<$REDIS_PASS> python <$PIPELINE_SRC_ROOT>/master.py -e -j wrfchem_pipeline.tasks.GetFNL -s $today >> /tmp/cron.out 2>&1
```

Content of `ndvi.sh`

```
#!/bin/bash
today=$(date +%Y%m%d)
REDIS_HOST=<$REDIS_HOST> REDIS_PASS=<$REDIS_PASS> python <$PIPELINE_SRC_ROOT>/master.py -e -j map_pipeline.tasks.downloadNDVI -s $today >> /tmp/cron.out 2>&1
```

Content of `mapNRT.sh`

```
#!/bin/bash
today=$(date +%Y%m%d)
REDIS_HOST=<$REDIS_HOST> REDIS_PASS=<$REDIS_PASS> python <$PIPELINE_SRC_ROOT>/master.py -e -j mapNRT_pipeline.tasks.downloadData -s $today >> /tmp/cron.out 2>&1
```

***Note:*** You should use a proper python interpreter that have all require dependencies (your proper virtual environment) if you setup multiple virtual environments.

## POPGIS Applications

To deploy POPGIS application, the following steps should be accomplished:

- Prepare a PostgreSQL database server, create and import application database from our database backup.
- Install and configure GeoServer
- Prepare a Microsoft Windows environment with Microsoft IIS version 10 Express, and Microsoft Visual Studio Community 2019 with 
    - ASP .NET and Web Tools 2019
    - ASP .NET Web Frameworks and Tools 2019
- Deploy project source code, adjust configurations and build solutions
- Configure application sites on ISS

### Prepare Database

In the sample scenario, PostgreSQL should be installed in server S3. For POPGIS Application, PostgreSQL 13 is recommended. The following commands is for installation of postgres database server and postgis extension

```
# apt install curl gpg gnupg2 software-properties-common apt-transport-https lsb-release ca-certificates
# curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc| gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
# echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | tee  /etc/apt/sources.list.d/pgdg.list
# apt update
# apt install postgresql-13 postgresql-client-13
# apt install postgis postgresql-13-postgis-3
```

Start postgresql server and make it autostart

```
# systemctl start postgres
# systemctl enable postgres
```

Create user, database and import initial data into the database

```
# su - postgres
$ createuser <$DB_USER> --pwprompt
$ createdb <$DB_NAME>
```

Grant privileges to the user on the database

```
$ psql
postgres=# grant all privileges on database <$DB_NAME> to <$DB_USER>;
postgres=# exit
```

Import database from provided backup 

```
$ psql <$DB_NAME> < backup_file.sql
```

### Install GeoServer

Make sure that Java SDK is installed on server S3. The following command will install Java on the server.

```
# apt install default-jdk
```

Unzip our GeoServer backup archive to a deployment location, e.g. /opt/geoserver

```
# unzip -d /opt/geoserver apom-geoserver-2.20.0.zip
```

### Prepare Microsoft Windows Environment

Server S4 in our sample scenario is for deployment of APOM_User and APOM_API application. Microsoft Windows operating system should be installed on the server. We recommend Microsoft Windows Server 2019 for the server. Make sure the following softwares are installed:

- Microsoft IIS version 10 Express (or higher)
- Microsoft Visual Studio Community 2019 with
    - ASP .NET and Web Tools 2019
    - ASP .NET Web Frameworks and Tools 2019

### Deploy Project Source Code

Checkout gitlab repository `apom_user` (See table 6) to a deploy location, e.g. `C:/Apom_Apps`, using `git clone`. Change `Web.config`, which is located in `Apom_User/Apom_User`, to configure _database connection_, _api service endpoint_, and _geoserver endpoint_.

For database connection, the configuration looks like:


    ...
    <connectionStrings>
        <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-Apom_User-20211115113655.mdf;Initial Catalog=aspnet-Apom_User-20211115113655;Integrated Security=True" providerName="System.Data.SqlClient" />
        <add name="DataApomAppConnection" connectionString="server=112.137.129.244;Port=5432;database=apom_app;user id=apom;password=apom!1111qaz;Integrated Security=True;" providerName="Npgsql" />
    </connectionStrings>
    ...


You should modify `connectionString` for `DataApomAppConnection` element.

For _api service endpoint_, the configuration looks like:

```
<add key="host_api" value="https://popgissvc.duckdns.org" />
<add key="host_geoserver" value="https://popgissvc.duckdns.org/geoserver" />
```

You should change value fields to suit your need.

We recommend to use a single domain name for all service and an Nginx reverse proxy for routing requests to appropriate services. Nginx configuration for such a proxy is provided in section [Reversed Proxy Configuration](#Reverse_Proxy_Configuration) for you reference. It is the actual configuration that we used in the current POPGIS system.

## Setup Community Site

We use Hugo and Docsy for easily publishing docs, news, posts, etc to community. To deploy community site, make sure that hugo is installed

```
# apt-get install hugo
```

Checkout `apom_web` GitLab repository with `git clone`. Move into the newly created directory and run

```
# cd apom_web
# hugo server
```

## Reverse Proxy Configuration

All services in POPGIS system are placed behind a reverse proxy using Nginx. Our current reverse proxy configuration is provided belows for reference

```
proxy_cache_path /var/www/mapcache1 keys_zone=mapcache1:10080m max_size=10g inactive=20160m use_temp_path=off;
server {
    root /var/www/html;
    set $originaddr https://fonts.googleapis.com;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name popgis.vnu.edu.vn;

    location /geoserver {
        gzip on;
        gzip_proxied any;
        proxy_pass http://112.137.129.248:8080;
    }
    location /api {
        proxy_pass http://112.137.129.248:8086;
    }
    location /theme {
        gzip on;
        gzip_proxied any;
        proxy_cache mapcache;
        proxy_pass http://112.137.129.248:8088;
    }
    location / {
        gzip on;
        gzip_proxied any;
        proxy_pass http://112.137.129.248:8088;
    }

    location /ggfont {
        rewrite /ggfont /css break;
        proxy_cache mapcache;
        proxy_pass https://fonts.googleapis.com;
    }
    location /ggfont2 {
        rewrite /ggfont2 /css2 break;
        proxy_cache mapcache;
        proxy_pass https://fonts.googleapis.com;
    }

    location /about {
        #rewrite ^/about/(.*)$ /$1 break;
        proxy_pass http://127.0.0.1:1313;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/popgis.vnu.edu.vn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/popgis.vnu.edu.vn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = popgis.vnu.edu.vn) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
    server_name popgis.vnu.edu.vn;
    listen 80;
    return 404; # managed by Certbot
}
```

# Notes for Customization and Extension

## POPGIS Processing Pipeline

### Source Code Structure

Directory layout of `apom_pipeline` repository is shown in the following diagram:

```
apom_pipeline
  ├ ...
  ├ constants.py
  ├ ...
  ├ wrfchem_pipeline
  │   ├ config.py
  |   ├ tasks.py
  |   └ scripts
  │      ├ ...
  │      ├ <stage code>
  │      ├ ...
  |
  ├ memNRT_pipeline
  │   ├ config.py
  |   ├ tasks.py
  |   └ scripts
  │      ├ ...
  │      ├ <stage code>
  │      ├ ...
  |
  ├ mem_pipeline
  │   ├ config.py
  |   ├ tasks.py
  |   └ scripts
  │      ├ ...
  │      ├ <stage code>
  │      ├ ...
  |
  ...
```

Level-1 subdirectories of the repository are corresponding to pipelines. Within each pipeline's directory, configurations of the pipeline is stored in `config.py`. Pipeline's stages are put inside `scripts` directory. Each stage should be runnable as a standalone application. A file whose name `tasks.py` in each pipeline's directory defines entry points for each stages. The POPGIS pipeline system will invoke stages via this file.

### Dependency graph

Dependency graph among stages are defined in `constants.py`, which is resided at the root or the directory. The graph is stored in `DEPENDENCY_GRAPH` constant and can be refered by `tasks.py`. Sample dependency declarations are shown belows.

```
DEPENDENCY_GRAPH = {
  'map_pipeline': {
    ...
  },
  "mapNRT_pipeline": {
    "downloadData": {
      "dependJobs": [],
      "nextJobs": [{"pipeline": "mapNRT_pipeline", "stage": "ImgPreprocess"}]
    },
    "ImgPreprocess": {
      "dependJobs": [{"pipeline": "mapNRT_pipeline", "stage": "downloadData"}],
      "nextJobs": [{"pipeline": "mapNRT_pipeline", "stage": "WRFChemProcessing"}]
    },
    "WRFChemProcessing": {
      "dependJobs": [{"pipeline": "mapNRT_pipeline", "stage": "ImgPreprocess"}, {"pipeline": "wrfchem_pipeline", "stage": "WRFpostprocessing"}],
      "nextJobs": [{"pipeline": "mapNRT_pipeline", "stage": "PM25predict"}]
    },
  },
  ...
}
```

### Adding a new pipeline and stages

To add a new pipeline, create a corresponding directory for it. Provide stages' source code in script directory. Note that stage should be runnable independently. Also, avoid hard coding variables and constants so that the stage will be portable. The stage should also support command line arguments so that it can take necessary inputs and configurations (date to run, input path, output path, etc.)

Configurations of all stages will be stored in `config.py`. In the file, you should at least define pipeline name in constant `PIPELINE`

After that, define entrypoints for stages in `tasks.py`. Each entrypoint of a stage is a exportable python function (its name should not start with `_` or `__`). The function should have one required argument which the date at with the stage is invoked. The function simply invokes desired stage via os shell. However, it should contain code to check whether it is eligible to run and code to invoke child stage. Template of an entrypoint for a stage is shown belows.

```
def <StageName>(startDate):
    stageName = '<StageName>'
    checkInfo = apis.can_run2(config.PIPELINE, stageName, startDate)
    if not checkInfo[0]:
        print("Cannot run because of " + str(checkInfo[1]))
        if checkInfo[1] == "Already run":
            master.enqueueNextJobs(config.PIPELINE, stageName, startDate)
        return
    rsl = []
    try:
        apis.upsert_pipeline_task(config.PIPELINE, stageName, startDate, 0)
        ########### code to invoke stage ############
        ...
        ############ END ###############
        if < success >:
            apis.upsert_pipeline_task(config.PIPELINE, stageName, startDate, 1)
            master.enqueueNextJobs(config.PIPELINE, stageName, startDate)
        else:
            apis.upsert_pipeline_task(config.PIPELINE, stageName, startDate, -1)
    except:
        print("Exception ........")
        apis.upsert_pipeline_task(config.PIPELINE, stageName, startDate, -2)

```

Finally, dependencies for new stages should be declared in `DEPENDENCY_GRAPH` in `constants.py`.

## POPGIS Web Application

### Source Code Structure

`apom_app` repository contains a Microsoft Visual Studio solution file `Apom_User.sln`. The file is located in `apom_app\Apom_User` directory. The solution file comprises of 3 projects 

- Apom_API: Application Rest API
- Apom_User: Application Frontend GUI
- And Repositories: ORM database mappings

Folder structure of the repository is shown belows

```
apom_app
  └ Apom_User
      ├ Apom_API
      |   ├ ...
      |   ├ Commons
      |   ├ Controllers
      |   ├ Models
      |   ├ Views
      |   ├ ...
      |   ├ Web.config
      |   ├ ...
      |   
      ├ Apom_User
      |   ├ ...
      |   ├ Commons
      |   ├ Controllers
      |   ├ Models
      |   ├ Views
      |   ├ ...
      |   ├ Web.config
      |   ├ ...
      ├ Repositories
      |   ├ ...
      |   ├ Models
      |   ├ Properties
      |   ├ Services
      |   ├ Constants.cs
      |   ├ ...
      ...
      │   
      └ Apom_User.sln
```

Configurations for projects (e.g. databese configurations, api endpoints, etc) are stored in `Web.config`. HTML/CSS/Javascript codes are ussually stored in `Views` directories. Backend logics such as controllers, and models can be found in `Controllers`, and `Models` respectively. Internationalization and translations can be found in `Apom_User\App_GlobalResources`

# Related Resources

[1]: Redis, https://redis.io/
[2]: Minio, https://min.io/
[3]: Python RQ, https://python-rq.org/
[4]: Anaconda - Getting Started, https://docs.anaconda.com/free/anacondaorg/user-guide/getting-started/
[5]: Nvidia, HPC SDK 20.11, https://docs.nvidia.com/hpc-sdk/archive/20.11/
[6]: GeoServer, https://docs.geoserver.org/
[7]: PostgreSQL, https://www.postgresql.org/docs/13/index.html
[8]: Hugo, https://gohugo.io/getting-started/quick-start/
[9]: Docsy, https://www.docsy.dev/

------------------ END OF DOCUMENT -------------------