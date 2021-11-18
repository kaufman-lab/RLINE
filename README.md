## 1. Implementation of RLINE in Linux

It is straight forward to implement RLINE. First, we need to download the source code of RLINE from https://www.cmascenter.org/r-line/, and compile it. The required inputs should be placed in the right spot and properly named, including traffic density, meteorological data and location for the prediction sites. Finally, run the executable will yield the desired results. The official documentation for RLINE is available at https://www.cmascenter.org/r-line/documentation/1.2/RLINE_UserGuide_11-13-2013.pdf. 

### 1.1 Download and compile RLINE executable 

We base our implementation on RLINE v1.2, which is updated in 2013. The source code is stored at https://github.com/kaufman-lab/RLINE/RLINE_source.zip. Note that the compilation scripts are also  contained in the zip file. Module GCC is required for compiling fortran source code. The following bash scripts illustrates how to compile RLINE.

```
[yunhanwu@deohs-brain ~]$ module load GCC/gcc-8.3.0 

```

Create folder structure,

```
[yunhanwu@deohs-brain ~]$ mkdir rline
[yunhanwu@deohs-brain ~]$ cd rline

```

Download RLINE source code,

```
[yunhanwu@deohs-brain rline]$ wget --no-check-certificate 
https://github.com/kaufman-lab/AERMET/blob/main/RLINE_source.zip?raw=true

mv RLINE_source.zip\?raw\=true RLINE_source.zip

```

Compile RLINE.

```
[yunhanwu@deohs-brain rline]$ cd RLINE_source
[yunhanwu@deohs-brain RLINE_source]$ MAKE_gfortran.sh

```


The compilation file is also attached. Note that the executable is named 'rline_out.out' in the last chuck, one could rename it if needed.

```

#!/bin/bash
echo "Compile RLINE using gfortran"

echo "Main program and declarations"
gfortran -c -Wall -fbounds-check -O1 Data_Structures.f90
gfortran -c -Wall -fbounds-check -O1 Line_Source_Data.f90
gfortran -c -Wall -fbounds-check -O1 RLINE_Main.f90

echo "Subprograms for reading inputs and setting up look-up table"
gfortran -c -Wall -fbounds-check -O1 Read_Line_Source_Inputs.f90
gfortran -c -Wall -fbounds-check -O1 Read_Met_Inputs.f90
gfortran -c -Wall -fbounds-check -O1 Read_Receptors.f90
gfortran -c -Wall -fbounds-check -O1 Read_Sources.f90
gfortran -c -Wall -fbounds-check -O1 Compute_File_Size.f90
gfortran -c -Wall -fbounds-check -O1 Create_Exp_Table.f90
gfortran -c -Wall -fbounds-check -O1 Expx.f90

echo "Subprograms for calculating concentrations and integrating point sources"
gfortran -c -Wall -fbounds-check -O1 Fill_Met.f90
gfortran -c -Wall -fbounds-check -O1 Compute_Met.f90
gfortran -c -Wall -fbounds-check -O1 Translate_Rotate.f90
gfortran -c -Wall -fbounds-check -O1 Numerical_Line_Source.f90
gfortran -c -Wall -fbounds-check -O1 Point_Conc.f90
gfortran -c -Wall -fbounds-check -O1 Meander.f90
gfortran -c -Wall -fbounds-check -O1 Polyinterp.f90
gfortran -c -Wall -fbounds-check -O1 Sigmay.f90
gfortran -c -Wall -fbounds-check -O1 Sigmaz.f90
gfortran -c -Wall -fbounds-check -O1 MOST_Wind.f90
gfortran -c -Wall -fbounds-check -O1 Effective_Wind.f90

echo "Subprograms for calculating concentrations using an analytical solution"
gfortran -c -Wall -fbounds-check -O1 Analytical_Line_Source.f90
gfortran -c -Wall -fbounds-check -O1 Analytical_Line_Parallel.f90

echo "Subprograms for calculating source displacements due to roadway features"
gfortran -c -Wall -fbounds-check -O1 Barrier_Displacement.f90
gfortran -c -Wall -fbounds-check -O1 Depressed_Displacement.f90

echo "Subprograms for writing output files and deallocating arrays"
gfortran -c -Wall -fbounds-check -O1 Write_Hourly_All.f90
gfortran -c -Wall -fbounds-check -O1 Write_Hourly_by_Month.f90
gfortran -c -Wall -fbounds-check -O1 Write_Daily_Ave.f90
gfortran -c -Wall -fbounds-check -O1 Deallocate_arrays.f90

gfortran -o rline_out.out Fill_Met.o Translate_Rotate.o Compute_File_Size.o
 Sigmay.o Line_Source_Data.o RLINE_Main.o Meander.o Compute_Met.o
  Numerical_Line_Source.o Analytical_Line_Source.o Analytical_Line_Parallel.o 
  Point_Conc.o Polyinterp.o Read_Line_Source_Inputs.o Read_Met_Inputs.o 
  Read_Receptors.o Read_Sources.o Data_Structures.o Sigmaz.o Write_Hourly_All.o 
  Write_Daily_Ave.o Write_Hourly_by_Month.o MOST_Wind.o Effective_Wind.o 
  Create_Exp_Table.o Expx.o Barrier_Displacement.o 
  Depressed_Displacement.o Deallocate_arrays.o

rm *.o *.mod

```



### 1.2 Prepare the required data

In the folder RLINE_source, we specify input files names in the control file named Line_Source_Inputs.txt file. We could modify RLINE configuration options in this file as well. Here we adopt everything as default except for whether to use analytical solution, which speeds up run time, but is less accurate. Since we have huge amount of traffic data, we use the analytical option to shorten computation time.

We then describe how to obtain the required input files.

1. **Met_Example.sfc**: This file is the meteorological data obtained from AERMET model. 
2. **Source_Example.txt**: Here we provide the traffic density on road segments in terms of AADT. Each row represents one segment. Features including starting/ending points will be put as columns in this file. in another Rmarkdown file, we will detail how to obtain such information from our traffic model. 
3. **Receptor_Example.txt**: Locations of the receptors.


```
User control file for RLINEv1_2
Source File Name
'Source_Example.txt'
Input Emiss can be in AADT or g/m (see user guide)
--------------------------------------------------
Receptor File Name
'Receptor_Example.txt'
--------------------------------------------------
Input Met File
'Met_Example.sfc'
--------------------------------------------------
Receptor Output File
'Output_Example_Numerical.csv'
--------------------------------------------------
Error_Limit (suggested 1.0e-03)
1.0e-03
--------------------------------------------------
Ratio of displacement height to roughness length (fac_dispht)
5.0
--------------------------------------------------
--- OUTPUT OPTION(S) BELOW: ----------------------
--------------------------------------------------
(1) Include concentrations from ['M'] Meander ONLY, ['P'] Plume ONLY, ['T'] Total = Plume+Meander
'T'
--------------------------------------------------
(2) Outout daily 24-hour averages? ('Y'/'N')
'Y'
--------------------------------------------------
(3) ['M'] Monthly Output Files, ['N'] No Hourly Files, ['A'] All hourly in one file
'M'
--------------------------------------------------
(4) Supress source/receptor proximity warnings? ('Y'/'N')
'Y'
--------------------------------------------------
--- BETA OPTION(S) BELOW: ------------------------
--------------------------------------------------
(1) Use analytical solution ('Y'/'N'), speeds up run time, but less accurate
'Y'
--------------------------------------------------
(2) Use barrier and depressed roadway algorithms? ('Y'/'N')
'N'
--------------------------------------------------
(3) Use non-zero roadwidth? ('Y'/'N')Lane width [m]
'N' 3.6
--------------------------------------------------


```

### 1.3 Run RLINE

After everything in place, we simply run the executable to get the results. Note that we need to convert the compiles source code as an executable using the following command. 

```
[yunhanwu@deohs-brain RLINE_source]$ ./rline_out.out # making output

```

Depends on the option that user sets, RLINE will produce hourly and daily-average output file. They are all in .csv format.
