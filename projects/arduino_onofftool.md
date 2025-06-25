---
title: Arduino - OnOffStress Test Tool
parent: Projects Achievement
layout: default
nav_order: 4
---

# Project Description
## Sovle what problems
- reduce RD testing time, optimize RD resource
- original testing method will need RD sit next to the M/B for hours
- it will be perfect if there is a automatic machine to improve this problem

## OnOffStress Box Featrues
- control AC/DC power automatically
- control ACPI power state: S0, S5, G3, powerbutton-trigger 
- integrated with GUI App for user input testing parameters
- high stability for testing 9999 cycles
- modulize componets for maitain easily
- high scalability for add new features
- low cost under NTD 1000

## Tool Photos
- Tool Frontside
![OnOffTool_45degreefrontside](https://github.com/user-attachments/assets/be7ee6bd-e540-4364-94c7-764f773c20ea)

- Tool Open case 
![OnOffToolopencase](https://github.com/user-attachments/assets/aeede8fc-1feb-4881-aa67-04768b8b6f0a)

- Tool Internal view
![OnOffTool_Internal](https://github.com/user-attachments/assets/be2cdd5a-a523-400f-8e1d-dc2058a31320)

## How it works
- Connect OnOffStress box and target M/B, then power-on both of them
![envsetup](https://github.com/user-attachments/assets/3adfb6c8-637b-4924-afab-2d149cf6f1dd)

- Open Testing App and set testing parameters, then start testing cycles
![starttesting](https://github.com/user-attachments/assets/4119b1f9-b100-4ccb-adc2-b197fa159399)


# HW document
## Components
![image](https://github.com/user-attachments/assets/033f863f-015e-4307-86eb-2081643b503e)

## Block diagram
![image](https://github.com/user-attachments/assets/9b692f67-109f-489e-8580-e9eb8ab6cdf7)

## Schematic
![image](https://github.com/user-attachments/assets/f3d1f3aa-b4e2-4d6a-8a1a-e562011364c4)

# SW document
## MCU code
- Please refer to github repository for more detailed code: [Arduino_OnOffStress-Test-Tool](https://github.com/OwenDuckLee/Arduino_OnOffStress-Test-Tool.git)
     
