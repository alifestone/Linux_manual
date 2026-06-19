# Python 虛擬環境建置
## conda
**建立**
```bash
conda create --name <name>
```
<br>

**啟用**<br>
```bash
# bash
conda activate <name>
```
```fish
# fish
source ~/miniforge3/etc/fish/conf.d/conda.fish

conda activate <name>
```
<br>

**列舉**
```bash
# list all installed package
conda list
# list specify installed package
conda list --name <package_name>

# list env
conda info --envs
```

**reference：** https://docs.conda.io/projects/conda/en/stable/commands/index.html
