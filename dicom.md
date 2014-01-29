---
title: DICOM to STL
layout: default
---
[DICOM To STL conversion tool](http://code.google.com/p/dicomtostl)  - is a command line utility which takes DICOM images (slice/tomography scans) and connect them in one 3D model in STL format. It is based on DCMTK library to parse DICOM format and uses Marching Cubes technique to build ISO Surface.  
It can be compiled for the Linux but CMake scripts should be little updated.  

|<img src="{{site.url}}/images/dicom/scan1.png" width="300" height="200"/>|<img src="{{site.url}}/images/dicom/scan2.png" width="300" height="200"/>|
--|--
|<img src="{{site.url}}/images/dicom/scan3.png" width="300" height="200"/>|<img src="{{site.url}}/images/dicom/model3d.png" width="300" height="200"/>|

 