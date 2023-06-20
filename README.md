# Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry

**Contacts:** bauer@bwh.harvard.edu, sophia.pells@umassmed.edu

Table of contents:
```diff
- 1. Objective and Background
- 2. Usage in GATE
```

## 1. Objective and Background

This directory provides an example of a GATE simulation for internal dosimetry calculations for a <sup>177</sup>Lu-DOTATATE therapy. This example has been written for GATE version 9. The patient source and attenuation images are available here https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation/blob/main/patient_LuDOTATATE_phantoms.zip

See  https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation#readme  for more information on the phantoms. 

The attenuation model: patient15_LuDOTATATE_attn.i33 <br />
The source model: patient15_LuDOTATATE_src.i33

The header files are the corresponding .h33 files. 

A screenshot of each (attn left and source right) is shown below 
![screenshot_patientPhantom](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry/assets/55833314/74ff5d8f-e3b7-45d3-88ad-d80d50738cd0)

 
  
  ## 2. Usage in GATE
  
 An example macro for a dose simulation for the photon emission, beta emission and monoenergetic electron emission of <sup>177</sup> is provided.  This section provides discusses some components of the macros in more detail. 
  
  The first step is to define the world and set the path to the file which defines all materials in the simulation: 
 ```ruby 
############################################################## 
# Materials definition
/gate/geometry/setMaterialDatabase /path/to/GateMaterials.db

############################### WORLD ################################
/gate/world/geometry/setXLength 150. cm 
/gate/world/geometry/setYLength 150. cm 
/gate/world/geometry/setZLength 260. cm 
/gate/world/vis/forceWireframe
/gate/world/vis/setVisible 0
/gate/world/vis/setColor green
/gate/world/setMaterial Air
```
 
  No scanner definition in needed to perform dose simulations in GATE. 
  
  ### Voxelised phantom definition
   ```ruby 
  ############################### Phantom ###############################
/gate/world/daughters/name voxelPhantom
/gate/world/daughters/insert ImageNestedParametrisedVolume 
/gate/voxelPhantom/geometry/setImage ./path/to/patient15_LuDOTATATE_attn.h33
/gate/voxelPhantom/geometry/setRangeToMaterialFile ./path/to/ID2mat.txt
/gate/voxelPhantom/placement/setTranslation  0. 0. 0. cm
  ```
 ID2mat.txt is the file which converts voxel values in the image to materials in your GateMaterials.dB file.  Note that it is not necesdsary to use the `attachPhantomSD` command for dose simulations. For the patient phantom given here, ID2mat.txt was as follows: 
```ruby
-2000 100 Air
100 220 CompressedAir
220 500 Lung
500 950 AdiposeTissue
950 1000 Muscle
1000 1150 SoftTissue
1120 2150 Bone
3000 3050 CarbonFiber
```
  The first and second column state the range of voxel values which correspond to the given material. The thirs column must match yhe name of a material defined in your materials.dB file. Tissue densities and compositions can be found in (for example) the supplemental matieral of ICRP Publication 110. 
  
  ### Voxelised source definition
  
  This example is for **only the beta** emissions of <sup>177</sup>Lu. First we must insert a voxel source, named VS_beta here: 
  
  ```ruby 
  # Add new source
/gate/source/addSource VS_beta voxel
/gate/source/VS_beta/reader/insert image
/gate/source/VS_beta/imageReader/translator/insert range
/gate/source/VS_beta/imageReader/rangeTranslator/readTable ./path/to/range_translator_file.dat
/gate/source/VS_beta/imageReader/rangeTranslator/describe 1
/gate/source/VS_beta/imageReader/readFile ./path/to/patient15_LuDOTATATE_src.h33
```
 where range_translator_file.dat specifies the region of the image ./path/to/patient15_LuDOTATATE_src.h33 to add source to and the activity (Bq/voxel). For example, to set an activity of 34 MBq to the spleen (with 1535 voxels), the file range_translator_file.dat would look like 
  
  ```ruby
  1
  16000.0	16000.0 22280.13	
  ```
The first line specifies that we want to set 1 active region. The second line specifies that all voxels of the source image with values between 16000 and 16000 will have an activity of 22280.13	Bq. <sup>177</sup>Lu has a 100% beta decay branch, so we do not need to scale the activity to account for the decay branch. 
  
  By default, the attenuation map will be centered on their geometric center, bur source phantoms are positoned such that their bottom left corner is at (0,0,0). Therefore we need to shift the voxel source by half its size in each axis to line up with the attenuation phantom. 
  
  ```ruby 
  /gate/source/VS_beta/setPosition -210.9402 -136.7205 -198.882 mm
  ```
  
  Next we set the decay data and emissions. Here a histogram of beta emissions has been created based on the DECDATA dasabase avaiable through ICRP Publication 107 (2008): https://www.icrp.org/publication.asp?id=ICRP%20Publication%20107
  
  ``` ruby
  # Define angular distribution of source
/gate/source/VS_beta/gps/ang/type iso

# Force unstable
/gate/source/VS_beta/setForcedUnstableFlag  true 
# Half life is 6.647 days 
/gate/source/VS_beta/setForcedHalfLife 574301 s

/gate/source/VS_beta/gps/particle    e-
/gate/source/VS_beta/gps/ene/type  Arb 
/gate/source/VS_beta/gps/hist/type    arb

/gate/source/VS_beta/gps/ene/min	0.0 MeV 
/gate/source/VS_beta/gps/ene/max	0.4978 MeV 

# Verbosity (good to initially uncomment to check source output is as expected)
#/gate/source/VS_beta/gps/verbose 2
# ---------------------------------------------------- #
/gate/source/VS_beta/gps/hist/point 0 0.0119193
/gate/source/VS_beta/gps/hist/point 0.0001 0.0119152
/gate/source/VS_beta/gps/hist/point 0.00011 0.0119131
/gate/source/VS_beta/gps/hist/point 0.00012 0.0119131
/gate/source/VS_beta/gps/hist/point 0.00013 0.0119131
/gate/source/VS_beta/gps/hist/point 0.00014 0.0119131
/gate/source/VS_beta/gps/hist/point 0.00015 0.011911
/gate/source/VS_beta/gps/hist/point 0.00016 0.011911
/gate/source/VS_beta/gps/hist/point 0.00018 0.011911
/gate/source/VS_beta/gps/hist/point 0.0002 0.0119089
/gate/source/VS_beta/gps/hist/point 0.00022 0.0119089
/gate/source/VS_beta/gps/hist/point 0.00024 0.0119068
/gate/source/VS_beta/gps/hist/point 0.00026 0.0119068
/gate/source/VS_beta/gps/hist/point 0.00028 0.0119047
/gate/source/VS_beta/gps/hist/point 0.0003 0.0119047
/gate/source/VS_beta/gps/hist/point 0.00032 0.0119026
/gate/source/VS_beta/gps/hist/point 0.00036 0.0119005
/gate/source/VS_beta/gps/hist/point 0.0004 0.0118985
/gate/source/VS_beta/gps/hist/point 0.00045 0.0118964
/gate/source/VS_beta/gps/hist/point 0.0005 0.0118922
/gate/source/VS_beta/gps/hist/point 0.00055 0.0118901
/gate/source/VS_beta/gps/hist/point 0.0006 0.011888
/gate/source/VS_beta/gps/hist/point 0.00065 0.0118859
/gate/source/VS_beta/gps/hist/point 0.0007 0.0118817
/gate/source/VS_beta/gps/hist/point 0.00075 0.0118797
/gate/source/VS_beta/gps/hist/point 0.0008 0.0118776
/gate/source/VS_beta/gps/hist/point 0.00085 0.0118734
/gate/source/VS_beta/gps/hist/point 0.0009 0.0118713
/gate/source/VS_beta/gps/hist/point 0.001 0.0118671
/gate/source/VS_beta/gps/hist/point 0.0011 0.0118609
/gate/source/VS_beta/gps/hist/point 0.0012 0.0118546
/gate/source/VS_beta/gps/hist/point 0.0013 0.0118504
/gate/source/VS_beta/gps/hist/point 0.0014 0.0118441
/gate/source/VS_beta/gps/hist/point 0.0015 0.01184
/gate/source/VS_beta/gps/hist/point 0.0016 0.0118337
/gate/source/VS_beta/gps/hist/point 0.0018 0.0118233
/gate/source/VS_beta/gps/hist/point 0.002 0.0118128
/gate/source/VS_beta/gps/hist/point 0.0022 0.0118024
/gate/source/VS_beta/gps/hist/point 0.0024 0.0117919
/gate/source/VS_beta/gps/hist/point 0.0026 0.0117815
/gate/source/VS_beta/gps/hist/point 0.0028 0.011771
/gate/source/VS_beta/gps/hist/point 0.003 0.0117585
/gate/source/VS_beta/gps/hist/point 0.0032 0.0117481
/gate/source/VS_beta/gps/hist/point 0.0036 0.0117272
/gate/source/VS_beta/gps/hist/point 0.004 0.0117063
/gate/source/VS_beta/gps/hist/point 0.0045 0.0116792
/gate/source/VS_beta/gps/hist/point 0.005 0.011652
/gate/source/VS_beta/gps/hist/point 0.0055 0.0116249
/gate/source/VS_beta/gps/hist/point 0.006 0.0115998
/gate/source/VS_beta/gps/hist/point 0.0065 0.0115726
/gate/source/VS_beta/gps/hist/point 0.007 0.0115455
/gate/source/VS_beta/gps/hist/point 0.0075 0.0115183
/gate/source/VS_beta/gps/hist/point 0.008 0.0114933
/gate/source/VS_beta/gps/hist/point 0.0085 0.0114661
/gate/source/VS_beta/gps/hist/point 0.009 0.011439
/gate/source/VS_beta/gps/hist/point 0.01 0.0113868
/gate/source/VS_beta/gps/hist/point 0.011 0.0113325
/gate/source/VS_beta/gps/hist/point 0.012 0.0112802
/gate/source/VS_beta/gps/hist/point 0.013 0.0112259
/gate/source/VS_beta/gps/hist/point 0.014 0.0111883
/gate/source/VS_beta/gps/hist/point 0.015 0.0111487
/gate/source/VS_beta/gps/hist/point 0.016 0.011109
/gate/source/VS_beta/gps/hist/point 0.018 0.0110275
/gate/source/VS_beta/gps/hist/point 0.02 0.010944
/gate/source/VS_beta/gps/hist/point 0.022 0.0108625
/gate/source/VS_beta/gps/hist/point 0.024 0.010779
/gate/source/VS_beta/gps/hist/point 0.026 0.0106975
/gate/source/VS_beta/gps/hist/point 0.028 0.010614
/gate/source/VS_beta/gps/hist/point 0.03 0.0105305
/gate/source/VS_beta/gps/hist/point 0.032 0.0104469
/gate/source/VS_beta/gps/hist/point 0.036 0.0102798
/gate/source/VS_beta/gps/hist/point 0.04 0.0101107
/gate/source/VS_beta/gps/hist/point 0.045 0.0098976
/gate/source/VS_beta/gps/hist/point 0.05 0.0096867
/gate/source/VS_beta/gps/hist/point 0.055 0.0094736
/gate/source/VS_beta/gps/hist/point 0.06 0.0092606
/gate/source/VS_beta/gps/hist/point 0.065 0.0090497
/gate/source/VS_beta/gps/hist/point 0.07 0.0088387
/gate/source/VS_beta/gps/hist/point 0.075 0.0086278
/gate/source/VS_beta/gps/hist/point 0.08 0.0084189
/gate/source/VS_beta/gps/hist/point 0.085 0.0082122
/gate/source/VS_beta/gps/hist/point 0.09 0.0080075
/gate/source/VS_beta/gps/hist/point 0.1 0.0076065
/gate/source/VS_beta/gps/hist/point 0.11 0.007218
/gate/source/VS_beta/gps/hist/point 0.12 0.0068421
/gate/source/VS_beta/gps/hist/point 0.13 0.006487
/gate/source/VS_beta/gps/hist/point 0.14 0.0061529
/gate/source/VS_beta/gps/hist/point 0.15 0.0058417
/gate/source/VS_beta/gps/hist/point 0.16 0.0055576
/gate/source/VS_beta/gps/hist/point 0.18 0.0050814
/gate/source/VS_beta/gps/hist/point 0.2 0.0046428
/gate/source/VS_beta/gps/hist/point 0.22 0.0041959
/gate/source/VS_beta/gps/hist/point 0.24 0.0037448
/gate/source/VS_beta/gps/hist/point 0.26 0.0032957
/gate/source/VS_beta/gps/hist/point 0.28 0.002855
/gate/source/VS_beta/gps/hist/point 0.3 0.0024248
/gate/source/VS_beta/gps/hist/point 0.32 0.002014
/gate/source/VS_beta/gps/hist/point 0.36 0.0012744
/gate/source/VS_beta/gps/hist/point 0.4 0.0006861
/gate/source/VS_beta/gps/hist/point 0.45 0.0001793
/gate/source/VS_beta/gps/hist/point 0.4978 0

/gate/source/VS_beta/gps/hist/inter Lin
/gate/source/list
  ```
  
  Each `/hist/point` gives an energy (MeV) and intensity of the continuous beta histrogram. The line `/gate/source/VS_beta/gps/hist/inter Lin` specifies how interpolation between points in the histogram is performed. 
  The source  must be defined **after** the `/gate/run/initialize` line in the macro. 
  
  
  ### Dose actors
  
  The dose actor must be defined **before** the `/gate/run/initialize` line in the macro. 
   
  ```ruby
############################## DOSE ACTOR ##############################
/gate/actor/addActor DoseActor           totEdep

/gate/actor/totEdep/attachTo             voxelPhantom
/gate/actor/totEdep/stepHitType     	 random
/gate/actor/totEdep/setPosition          0 0 0 mm

# Set aliasing for size and resolution of dose actor
# Needs to match size of images used for geometry and source
# x = 108 * 3.9063 mm
# y = 70 * * 3.9063 mm
# z = 90 * 4.4196 mm
/gate/actor/totEdep/setSize  421.8804 273.441 397.764 mm
/gate/actor/totEdep/setResolution 108 70 90

# Set desired actors
# Record Edep here and its uncertainty
# squared Edep can be useful uncertainty calculations
gate/actor/totEdep/enableDose             false
/gate/actor/totEdep/enableEdep             true
/gate/actor/totEdep/enableUncertaintyDose  false
/gate/actor/totEdep/enableUncertaintyEdep  true
/gate/actor/totEdep/enableSquaredDose      false
/gate/actor/totEdep/enableSquaredEdep      true
/gate/actor/totEdep/enableNumberOfHits    false
/gate/actor/totEdep/save                  output_dir/output_name.hdr
/gate/actor/totEdep/saveEveryNSeconds      600

# Track stats actor too
/gate/actor/addActor SimulationStatisticActor     statsActor
/gate/actor/statsActor/save                       output_dir/output_name.txt

##############################
  ```
  The dose actor will create interfile images with the same size and resolution as our input source image. The output is 32 bit float. The Edep image has units of MeV and the uncertainty is the **relative** statistical uncertainty. The squared file is useful for calculating the uncertainty if multiple files are combined. 
  
  Example output Edep image for betas(left) and photons (right):
  ![exampleOutput_EdepSpleen_betas_photons](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry/assets/55833314/00435f52-d51b-4f35-9e76-3171f13551df)

