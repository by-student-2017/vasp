#! /bin/csh -f
# VASP run command
#
# user settting
set vasp_adress = $HOME'/vasp/vasp.5.4.1/bin/vasp_std'
set pseudo_potential_adress = $HOME'/vasp/potpaw_PBE'
#set standard_input_adress = $HOME'/Desktop/Link to vasp_work/standard_input_files'
set __mpirun__ = "mpirun"
#if you get libraries form Ubuntu Software center, I reccommend below,
#set __mpirun__ = "mpirun.openmpi"
#
# Automatically setting
set kpoint_mesh = $1
set kpoint_mesh_name = $2
setenv OMP_NUM_THREADS 1
set node = `grep 'physical id' /proc/cpuinfo | sort -u | wc -l`
set num_core = `grep 'core id' /proc/cpuinfo | sort -u | wc -l`
echo ""
set DATE = `date`
echo $DATE
echo "Automatically setting environments"
echo "------------------------------------------------------------"
echo "OpenMP threads = 1"
echo "automatic setting Parallel calculation (MPI) = "$num_core" core"" / ""$node"" node"
set NPAR = `echo 4-"sqrt($num_core)" | bc`
echo "automatic setting NPAR = "$NPAR
#sed 's/#NPAR/NPAR = '${NPAR}'/g' "${standard_input_adress}/INCAR_scf_template" > "${standard_input_adress}/INCAR_scf"
echo "------------------------------------------------------------"
echo $vasp_adress
echo $pseudo_potential_adress
#echo $standard_input_adress
echo "------------------------------------------------------------"
#
# current adress setting
set address = `pwd`
echo "$address"
set DATE = `date`
#
# get cif data
set cif_list=`ls *.cif`
foreach cif_name ( $cif_list )
  # remove extension
  set cif_folder_name = `echo $cif_name:r`
  mkdir ${cif_folder_name}
  cp "$address/$cif_name" "$address/${cif_folder_name}/$cif_name"
end
echo "include"
set list=`find . -maxdepth 1 -mindepth 1 -type d | awk -F/ '{print $NF}'`
echo $list
echo "------------------------------------------------------------"
#
# loop
foreach folder_name_list ( $cif_list )
  set folder_name = `echo $folder_name_list:r`
  set DATE = `date`
  #
  echo ""
  echo "vasp calculation start: ""$folder_name"
  echo ""
  date
  #
  cd "$address/$folder_name"
  #
  if ( -f disp.yaml ) then
    echo "already disp.yaml"
  else
    echo "run cif2cell"
    cif2cell -p vasp --setup-all --vasp-format=5 --vasp-encutfac=1.0 --vasp-pseudo-libdr="${pseudo_potential_adress}" --vasp-cartesian-lattice-vectors -f *.cif
    cp INCAR INCAR_cif2cell
    cp POSCAR POSCAR_temp
    mv KPOINTS KPOINTS_
    # change volume, (defaut value -5 -4 -3 -2 -1 0 1 2 3 4 5)
    foreach no ( -5 -4 -3 -2 -1 0 1 2 3 4 5 )
      #
      # scf calculation
      if ( ${kpoint_mesh} == "-k" ) then
        if ( ${kpoint_mesh_name} == "" ) then
          mv KPOINTS_ KPOINTS
        else
          cp $address/${kpoint_mesh_name} KPOINTS
        endif
      endif
      date
      set DATE = `date`
      echo "scf calculation"
      echo "${no}, scf calculation: "$DATE
      #
      # INCAR
      echo "NPAR = "$NPAR > INCAR
      echo "   PREC = Accurate" >> INCAR
      echo "#  ENCUT = 500    " >> INCAR
      echo " ISMEAR = 0; SIGMA = 0.1" >> INCAR
      echo "  LREAL =  Auto   " >> INCAR
      set volume_times = `awk -v p="${no}" 'NR==2 {print $1*(100-p)/100}' POSCAR_temp`
      sed 1,2d POSCAR_temp > POSCAR_temp2
      echo qha calculation > POSCAR
      echo ${volume_times} >> POSCAR
      cat POSCAR_temp2 >> POSCAR
      rm -f -r POSCAR_temp2
      ${__mpirun__} -np $num_core $vasp_adress # vasp calculation
      #
      # volume and total energy
      grep "volume of cell" OUTCAR | tail -1 > Volume_temp
      set Volume = `awk '{print $5}' Volume_temp`
      grep "TOTEN" OUTCAR | tail -1 > TOTEN_temp
      set TOTEN  = `awk '{print $5}' TOTEN_temp`
      echo ${Volume}"   "${TOTEN} >> e-v.dat
      cp OUTCAR OUTCAR-${no}
      cp POSCAR POSCAR-${no}
      #
      #
      # dftp calculation
      if ( ${kpoint_mesh} == "-k" ) then
        mv KPOINTS KPOINTS_
      endif
      date
      set DATE = `date`
      echo "dfpt calculation"
      echo "${no}, dfpt calculation: "$DATE
      #
      # INCAR
      echo "   PREC = Accurate" > INCAR
      echo "#  ENCUT = 500    " >> INCAR
      echo " IBRION = 8       " >> INCAR
      echo "  EDIFF = 1.0e-08 " >> INCAR
      echo "  IALGO = 38      " >> INCAR
      echo " ISMEAR = 0; SIGMA = 0.1" >> INCAR
      echo "  LREAL = .FALSE. " >> INCAR
      echo "ADDGRID = .TRUE.  " >> INCAR
      echo "  LWAVE = .FALSE. " >> INCAR
      echo " LCHARG = .FALSE. " >> INCAR
      #
      cp POSCAR POSCAR_backup
      mv POSCAR POSCAR-unitcell
      phonopy -d --dim="2 2 2" -c POSCAR-unitcell
      mv SPOSCAR POSCAR
      ${__mpirun__} -np $num_core $vasp_adress # vasp calculation
      phonopy --fc vasprun.xml
      #
      # phonopy, thermal_properties.yaml
      if ( -f mesh.conf ) then
        echo "use mesh.conf"
      else
        echo "automatically make mesh.conf file"
        echo "DIM = 2 2 2           " >  mesh.conf
        echo "MP = 16 16 16         " >> mesh.conf
        echo "FORCE_CONSTANTS = READ" >> mesh.conf
        echo "TPROP = .TRUE.        " >> mesh.conf
        echo "TMIN  = 0             " >> mesh.conf
        echo "TMAX  = 2000          " >> mesh.conf
        echo "TSTEP = 10            " >> mesh.conf
        endif
        phonopy --dim="2 2 2" -c POSCAR-unitcell -t -p mesh.conf -s
        cp thermal_properties.yaml thermal_properties.yaml-${no}
        set therm_prop_yaml = `cat therm_prop_yaml_name_list`
        echo "${therm_prop_yaml} thermal_properties.yaml-${no}" > therm_prop_yaml_name_list
      endif
    #
    end
  #
  endif
  #
  set therm_prop_yaml = `cat therm_prop_yaml_name_list`
  #rm -f -r therm_prop_yaml_temp
  # pressure, GPa units
  #phonopy-qha --sparse=14  --tmax=1400 --pressure=0.000101325 -p e-v.dat thermal_properties.yaml-{-{5..1},{0..5}} -s
  phonopy-qha --sparse=14  --tmax=1400 --pressure=0.000101325 -p e-v.dat ${therm_prop_yaml} -s
  #
  #phonopy-qha --sparse=14  --tmax=1400 --pressure=0.000101325 -p e-v.dat thermal_properties.yaml-{-{5..1},{0..5}}
  phonopy-qha --sparse=14  --tmax=1400 --pressure=0.000101325 -p e-v.dat ${therm_prop_yaml}
  #
  # change volume, (defaut value -5 -4 -3 -2 -1 0 1 2 3 4 5)
  #phonopy-qha --sparse=14 --tmax=1400 --pressure=0.000101325 -p e-v.dat thermal_properties.yaml--5 thermal_properties.yaml--4 thermal_properties.yaml--3 thermal_properties.yaml--2 thermal_properties.yaml--1 thermal_properties.yaml-0 thermal_properties.yaml-1 thermal_properties.yaml-2 thermal_properties.yaml-3 thermal_properties.yaml-4 thermal_properties.yaml-5 -s
  #
  # change volume, (defaut value -5 -4 -3 -2 -1 0 1 2 3 4 5)
  #phonopy-qha --sparse=14 --tmax=1400 --pressure=0.000101325 -p e-v.dat thermal_properties.yaml--5 thermal_properties.yaml--4 thermal_properties.yaml--3 thermal_properties.yaml--2 thermal_properties.yaml--1 thermal_properties.yaml-0 thermal_properties.yaml-1 thermal_properties.yaml-2 thermal_properties.yaml-3 thermal_properties.yaml-4 thermal_properties.yaml-5 &
  #
  phonopy-qha -b e-v.dat
  phonopy-qha --eos='birch_murnaghan' -b e-v.dat
  rm -f -r POSCAR_temp
  cd $address
end
#
