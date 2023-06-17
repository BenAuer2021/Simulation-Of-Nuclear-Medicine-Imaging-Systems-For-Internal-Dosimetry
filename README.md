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

The voxelised phantom model is based on a low-dose CT of a female <sup>177</sup>Lu-DOTATATE patient from the Christie NHS Foundation Trust, Manchester, UK. 
A model for attenuation and source definition is provided. Both are unsigned 16bit images of size 108x70x90. 

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
