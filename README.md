# Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry

Table of contents:
```diff
- 1. Objective and Background
- 2. Patient phantom
- 3. Usage in GATE
```

## 1. Objective and Background

This directory provides an example of a GATE simulation for internal dosimetry calculations for a <sup>177</sup>Lu-DOTATATE therapy. 


## 2. Patient phantom

A voxelised phantom model  based on a low-dose CT of a female <sup>177</sup>Lu-DOTATATE patient The voxelised phantom model is from the Christie NHS Foundation Trust, Manchester, UK. Registration and organ/tumour segmentation was performed by Dr Emma Page at the Christie NHS Foundation Trust, Manchester, UK.

A model for attenuation and source definition is provided. Both are in interfile format with unsigned 16bits and size 108x70x90. 

The attenuation model: patient15_LuDOTATATE_attn.i33 <br />
The source model: patient15_LuDOTATATE_src.i33

The header files are the corresponding .h33 files. 

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

<p align="center">
  <img width="1191" alt="Patient phantom" src="![image](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry/assets/55833314/ad35bcf9-fd2d-4b8f-a907-31c462fb9596)"
</p>
  
  
  ## 3. Usage in GATE
  
  This section provides examples for absorbed dose calculations for the spleen of the patient phantom.
  
  No scanner definition in needed to perform dose simulations in GATE. 
  
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
