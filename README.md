# Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry

This directory provides an example of a GATE simulation for internal dosimetry calculations for a <sup>177</sup>Lu-DOTATATE therapy. 

Table of contents:
```diff
- 1. Objective and Background
- 2. Patient phantom
- 3. Usage in GATE
```

## 1. Patient phantom

The voxelised phantom model is based on a low-dose CT of a female <sup>177</sup>Lu-DOTATATE patient from the Christie NHS Foundation Trust, Manchester, UK. 
A model for attenuation and source definition is provided. Both are unsigned 16bit images of size 108x70x90. 

The attenuation model: patient15_LuDOTATATE_attn.i33
TGhe source model: patient15_LuDOTATATE_src.i33

<p align="center">
  <img width="1191" alt="Patient phantom" src="![image](https://github.com/BenAuer2021/Simulation-Of-Nuclear-Medicine-Imaging-Systems-For-Internal-Dosimetry/assets/55833314/ad35bcf9-fd2d-4b8f-a907-31c462fb9596)"
</p>
