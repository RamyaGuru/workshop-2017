# Installing Atomate
This documentation is meant as a supplemental to the [official install documentation](https://hackingmaterials.github.io/atomate/installation.html) 

# Prerequisites
1. Computing cluster that will let you run VASP
2. Working local Python 3.6 Anaconda install on that computer
3. Two MongoDB deployments running on [mLab](https://mlab.com) with one having only 
   an admin user and one with both an admin user and a read only user.
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
running `pip install -e .` or `python setup.py develop` if pip fails.  

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
>    scratch_dir: <<>SCRATCH_DIR> 
<<WORKER_NAME>> is simply a name for this worker. You can name it after the cluster if you plan to have multiple compute resources. 











