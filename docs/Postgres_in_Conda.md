#### Create conda environment
`conda create --name myenv`

#### Enter the environment
`conda activate myenv`

### Install postgresql via conda
`conda install -y -c conda-forge postgresql`

### Create a base database locally
`initdb -D mylocal_db`

### Now start the server modus/instance of postgres
`pg_ctl -D mylocal_db -l logfile start`

```
waiting for server to start.... done
server started
``` 
### Alter postgres user password
`psql -c "alter user postgres password 'password'"` 

