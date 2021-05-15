# Datumaro Feature List

* 本文件用于罗列datumaro commandline直接可用的工具
* 同时罗列出不可以用但是后续可能需要用到的工具

## 使用须知
* datumaro gihub: https://github.com/openvinotoolkit/datumaro
* 现datumaro工具在command line直接下载运行会有bug
* 于是我们就拉其代码到本地，并且使用用python 3.7.6以下的版本运行datumaro

## Datumaro feature 介绍

### 1. 数据集(dataset)创建

- 创建数据集，输入储图片的文件夹
  ```bash
  # This command allows to convert a dataset from one format into another:
  datum create -o <project/dir>
  ```

### 2. 数据集(dataset)转换

- 转换数据集
  ```bash
  # To create an unlabelled dataset from an arbitrary directory with images use ImageDir format:
  datum convert --input-format voc --input-path <path/to/voc/> \
              --output-format coco
  # or
  datum convert \
    -i <input path> \
    -if <input format> \
    -o <output path> \
    -f <output format> \
    -- [extra parameters for output format]
  ```

### 3. 项目(project)录入

- 通过数据集录入:
  ```bash
  # This command creates a Project from an existing dataset
  datum import \
      -i <dataset_path> \
      -o <project_dir> \
      -f <format>
  ```

### 4. 项目(project)创建

- 创建空项目:
  ```bash
  # This command creates a Project from scratch
  datum create \
      -o <project_dir>
  ```

### 5. 项目(project)添加或删除数据

- 往项目里添加或删除数据:
  ```bash
  # This command add or remove data from project
  datum add \
      path <path> \
      -p <project dir> \
      -f <format> \
      -n <name>

  datum remove \
      -p <project dir> \
      -n <name>
  ```
  
### 6. 项目(project)添加或删除数据

- 往项目里添加或删除数据:
  ```bash
  # This command add or remove data from project
  datum add \
      path <path> \
      -p <project dir> \
      -f <format> \
      -n <name>

  datum remove \
      -p <project dir> \
      -n <name>
  ```
  
### 7. 项目(project)更新

- 通过第二个项目的标注信息和第一个项目融合，于是产出第三个新的合并项目:
  ```bash
  # update annotations in the first_project with annotations from the second_project and save the result as merged_project
  datum merge \
      -p <project dir> \
      -o <output dir> \
      <other project dir>
  ```
  
### 8. 项目(project)输出(export)

- 提供一个项目，输出一个数据集，数据集的格式可以自己定义:
  ```bash
  # This command exports a Project as a dataset in some format
  datum export \
      -p <project dir> \
      -o <output dir> \
      -f <format> \
      -- [additional format parameters]
  ```
  
### 9. Get project info

- 查看项目细节:
  ```bash
  # This command outputs project status information
  datum info \
      -p <project dir>
  ```

### 10. Get project info

- 查看项目细节:
  ```bash
  # This command outputs project status information
  datum info \
      -p <project dir>
  ```
  
  
  
## model是否需要去和dataset一起展开inference
## 
  
  
  
  
  
  
