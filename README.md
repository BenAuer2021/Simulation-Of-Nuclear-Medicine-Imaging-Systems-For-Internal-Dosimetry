# Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry

Table of contents:
```diff
- 1. Objective and Background
- 2. Patient phantom
- 3. Usage in GATE
```

## 1. Objective and Background

This directory provides an example of a GATE simulation for internal dosimetry calculations for a <sup>177</sup>Lu-DOTATATE therapy. This example has been written for GATE version 9. 


## 2. Patient phantom

A voxelised phantom model  based on a low-dose CT of a female <sup>177</sup>Lu-DOTATATE patient The voxelised phantom model is from the Christie NHS Foundation Trust, Manchester, UK. Registration and organ/tumour segmentation was performed by Dr Emma Page at the Christie NHS Foundation Trust, Manchester, UK.

A model for attenuation and source definition is provided. Both are in interfile format with unsigned 16bits and size 108x70x90. The pixel size is 3.9063 mm and the slice thickness is  4.4196 mm. 

The attenuation model: patient15_LuDOTATATE_attn.i33 <br />
The source model: patient15_LuDOTATATE_src.i33

The header files are the corresponding .h33 files. 

A screenshot of each (attn left and source right) is shown below 
<p align="center">
  <img width="1191" alt="Patient phantom" src="![screenshot_patientPhantom](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry/assets/55833314/2e1d87cd-b8e8-4a93-842c-d2ac79c09c02)"
</p>

The attenuation map is based on Hounsfield units, apart from the patient bed which was manually segmented and set to voxel values of 3000 to allow its material to be specified. 
The range of values in the attenuation file is as follows: 
|     Min value |     Max value   |     Material   |
|---------------|-----------------|----------------|
| -2000         | 100             | Air            |
| 100           | 220             | Compressed Air |
| 220           | 500             | Lung           |
| 500           | 950             | Adipose Tissue |
| 950           | 1000            | Muscle         |
| 1000          | 1150            | Soft Tissue    |
| 1120          | 2150            | Bone           |
| 3000          | 3050            | Carbon Fibre   |

The source model defines the liver, spleen, left and right kidneys and three tumours with the following values: 
|     Region        |     Voxel value    |     Number   of source voxels    |
|-------------------|--------------------|----------------------------------|
|     Liver         |     15000          |     15381                        |
|     Spleen        |     16000          |     1535                         |
|     L   Kidney    |     17000          |     2052                         |
|     R   Kidney    |     18000          |     2493                         |
|     Tumour1       |     19000          |     93                           |
|     Tumour2       |     20000          |     55                           |
|     Tumour3       |     21000          |     90                           |

This permits activities to be specified for different regions with an activity range data file in GATE (see voxelised source below). 


  
  
  ## 3. Usage in GATE
  
  This section provides examples for absorbed dose calculations for the spleen of the patient phantom.
  
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
  Tissue densities and compositions can be found in (for example) the supplemental matieral of ICRP Publication 110. 
  
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
  
  By default, the attenuation map will be centered in the world but the voxel source gets centered to the top right corner of the world. Therefore we need to shift the voxel source by hal
  
  Next we set the decay data and emissions: 
  
  
  
  
  
  ### Dose actors
  
   
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
/gate/actor/totEdep/save                  *output_dir*/*output_name*.hdr
/gate/actor/totEdep/saveEveryNSeconds      600

# Track stats actor too
/gate/actor/addActor SimulationStatisticActor     statsActor
/gate/actor/statsActor/save                       *output_dir*/*output_name*.txt

##############################
  ```
