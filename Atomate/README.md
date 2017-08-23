# Installing Atomate
This documentation is meant as a supplemental to the [official install documentation](https://hackingmaterials.github.io/atomate/installation.html) 

# Prerequisites
1. Computing cluster that will let you run VASP
2. Working Python 3.6 Anaconda install on that computer
3. Two MongoDB deployments running on [mLab](https://mlab.com) with one having only 
   an admin user and one with both an admin user and a read only user.  
   *Note*: passwords will be stored in plain text, so make sure only you can view the directory they are installed to.
4. If you have intel compilers loaded, unload the module and instead use gcc. This is only necessary for install, not operation. Pymatgen's install will fail otherwise.

You will want your VASP pseudopotentials to be arranged as follows:

pseudopotentials   
├── POT_GGA_PAW_PBE   
│      ├── POTCAR.Ac.gz   
│      ├── POTCAR.Ac_s.gz   
│      ├── POTCAR.Ag.gz   
│      └── ...   
├── POT_GGA_PAW_PW91   
│      ├── POTCAR.Ac.gz   
│      ├── POTCAR.Ac_s.gz   
│      ├── POTCAR.Ag.gz   
│      └── ...   
└── POT_LDA_PAW   
|      ├── POTCAR.Ac.gz   
|      ├── POTCAR.Ac_s.gz   
|      ├── POTCAR.Ag.gz   
|      └── ...

# Prepare install folder
Create a Python environment for your install using conda.
e.g. `conda create --name atomate_env python=3`

In your environment folder create three folders labelled `codes`, `config`, and `logs`respectively.

Enter `source activate atomate_env` or whatever your environment name is to activate it. 
Python will now default to only using and installing modules specific to this environment.

# Installing Python packages
`cd` to your codes directory. Now, download all the packages you need from GitHub
***
`git clone https://www.github.com/materialsproject/fireworks.git`   
`git clone https://www.github.com/materialsproject/pymatgen.git`   
`git clone https://www.github.com/atztogo/phonopy.git`  
`git clone https://www.github.com/materialsvirtuallab/pymatgen-diffusion.git`   
`git clone https://www.github.com/materialsproject/pymatgen-db.git`   
`git clone https://www.github.com/materialsproject/custodian.git`   
`git clone https://www.github.com/hackingmaterials/atomate.git`  
***
Next, install the packages **in the order shown above** by going into each directory and 
running `pip install -e .` or `python setup.py develop` if pip fails. One exception may be the diffusion package, which atomate will install as part of its setup.

# Configuring FireWorks
Create the following in your config folder. Anything with <\<VARIABLE>> should be replaced with the actual variable.  
e.g. "<\<PORT>>": "11716" or <\<ADMIN_USERNAME>>: admin 
## db.json
This file configures the database where your results will be stored. The necessary information can be found in your mLab account. This JSON file requires string arguments for everything but the port and aliases, so leave the quotation marks in.
>{  
>    "host": "<\<HOSTNAME>>",  
>    "port": <\<PORT>>,  
>    "database": "<<DB_NAME>>",  
>    "collection": "tasks",  
>    "admin_user": "<<ADMIN_USERNAME>>",  
>    "admin_password": "<<ADMIN_PASSWORD>>",  
>    "readonly_user": "<<READ_ONLY_PASSWORD>>",  
>    "readonly_password": "<<READ_ONLY_PASSWORD>>",  
>    "aliases": {}  
>}

## my_fworker.yaml
This file controls the environment unique to each cluster, such as specifying the command to run vasp.
>name: <<WORKER_NAME>>  
>category: ''  
>query: '{}'  
>env:  
>    db_file: <<INSTALL_DIR>>/config/db.json  
>    vasp_cmd: <<VASP_CMD>>  
>    scratch_dir: <<SCRATCH_DIR>> 

<<WORKER_NAME>> is simply a name for this worker. You can name it after the cluster if you plan to have multiple compute resources.   
<<INSTALL_DIR>> refers to the absolute path of where you are installing atomate.  
<<VASP_CMD>> is the command you use to run vasp, e.g. `mpirun vasp` or `ibrun vasp_std`  
<<SCRATCH_DIR>> is exactly what it says. It can be set to null if you prefer to run where you submit jobs.

## my_launchpad.yaml
This file configures the actual database that controls the fireworks. The variables should be self-explanatory.   
Just remember this is for the database with only the admin account.
>host: <<HOSTNAME>>  
>port: <<PORT>>  
>name: <<DB_NAME>>  
>username: <<ADMIN_USERNAME>>  
>password: <<ADMIN_PASSWORD>>  
>ssl_ca_file: null  
>strm_lvl: INFO  
>user_indices: []  
>wf_user_indices: []

## my_qadapter.yaml
This file lets atomate confiure the queue system for your needs. 
>\_fw_name: CommonAdapter  
>\_fw_q_type: SLURM  
>rocket_launch: rlaunch -c <<INSTALL_DIR>>/config singleshot  
>nodes: 2  
>walltime: <<WALL_TIME>>  
>queue: normal  
>account: <<ACCOUNT>>  
>job_name: null  
>pre_rocket: null  
>post_rocket: null  
>logdir: <<INSTALL_DIR>>/logs  

The settings here are a basic configuration for SLURM schedulers like the one at Stampede2. 
"CommonAdapter" indicates we are using one of the built in queue systems, while the \_fw_q_type can be one of the following:
  * SLURM
  * PBS/Torque
  * SGE
  * IBM LoadLeveler  
<<WALL_TIME>> should be specified in minutes. e.g. 1440 for 24 hours. 
<<ACCOUNT>> is just the name of the account you will be billing.

## FW_config.yaml
This file is just to let FireWorks know where all your config files are located.
>CONFIG_FILE_DIR: <<INSTALL_DIR>>/config

## Finishing up configuring FireWorks
Now, you just need to add the following to your .bashrc with appropriate substitution and source it
`export FW_CONFIG_FILE=<<INSTALL_DIR>>/config/FW_config.yaml`
And run `lpad reset` to see if you can connect to the FireWorks server.

# Configure pymatgen
Create a file in your home directory named `.pmgrc.yaml`. Only the first flag here is necessary for VASP. The second is for if you want to use Materials Project resources, 
while the functional argument can be set to PS for PBEsol or other VASP supported functionals. If you don't need to query 
Materials Project and are fine with the default XC-functional in the POTCAR, just delete the lines.
>PMG_VASP_PSP_DIR: <<INSTALL_DIR>>/pps  
>PMG_MAPI_KEY: <<YOUR_API_KEY>>  
>DEFAULT_FUNCTIONAL: <<XC_FUNCTIONAL>>

# Run a test workflow
To test if everything has come together, you can run the following on the command line with your environment activated: 
`atwf add -l vasp -p wf_bandstructure -m mp-149`
(This test requires your Materials Project API key to be specified)
If it looks like it worked, check with `lpad get_wflows`  
It should return
>[  
>    {  
>        "state": "READY",  
>        "name": "Si--1",  
>        "created_on": "2015-12-30T18:00:00.000000",  
>        "states_list": "REA"  
>    },  
>]  
Then, navigate to where you want to run the calculations and run `qlaunch -r rapidfire`  
When the job finishes, you can run  
`mgdb query -c <<INSTALL_DIR>>/config/db.json --props task_id formula_pretty output.energy_per_atom`  

to see if it finished correctly.  <<INSTALL_DIR>> can be relative here.  
For debugging, you should look at the official documentation or post in the [Google group](https://groups.google.com/forum/#!forum/atomate)




