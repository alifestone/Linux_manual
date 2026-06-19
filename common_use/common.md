# Python 虛擬環境建置
## conda
**create env**
```bash
conda create --name <name>
```

**activate**
```bash
# bash
conda activate <name>
```
```fish
# fish
source ~/miniforge3/etc/fish/conf.d/conda.fish

conda activate <name>
```

**install**
```bash
conda install <package>
# or
pip install <package>
```

**list**
```bash
# list all installed package
conda list
# list specify installed package
conda list --name <package_name>

# list env
conda info --envs
```

**reference：** https://docs.conda.io/projects/conda/en/stable/commands/index.html
<br><br>

## uv
The official uv documentation already has a complete guide. It is highly recommended to read it.<br>
[link](https://docs.astral.sh/uv/guides/projects/)
<br><br>

## Python venv
**create**
```bash
python -m venv <name>
```

**activate**
```bash
# Linux or MacOS
source <name>/bin/activate
```

**install**
```bash
python -m pip install <name>(==<version>)
```

**list**
```bash
# list all installed package
python -m pip list
# list specify installed package
python -m pip show <package>

# find env name
tree . | grep activate # or any tool can list files
```

**reference：** https://docs.python.org/zh-tw/3/tutorial/venv.html
