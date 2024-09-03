# poetry
## 安装pipx
* [Pipx Installation](https://pipx.pypa.io/stable/installation/)
~~`conda create --prefix C:\conda_envs\pipx_env python=3.9`~~
~~`conda activate  C:\conda_envs\pipx_env`~~
~~`conda remove --prefix C:\conda_envs\pipx_env --all`~~
  + conda create命令的`--prefix`参数用于指定env的安装位置，
  + `--name`参数用于命名env，两个参数不兼容
```bash
# 在conda base env下，或其他全局环境下，安装pipx
# --user参数表示Python 会将包安装到用户级别的路径，而不是当前激活的虚拟环境路径中。
# 具体来说，--user 选项会将包安装到类C:\Users\<YourUsername>\AppData\Roaming\Python\Python39\site-packages 的目录中，
# 而不是虚拟环境的 site-packages 目录中。
python -m pip install --user pipx

# 把pipx的执行路径添加到系统环境变量中
pipx ensurepath
```
## 安装和更新poetry
* [Installation](https://python-poetry.org/docs/#installation)
  + **Poetry 应始终安装在专用的虚拟环境中，以将其与系统的其他部分隔离开来。在任何情况下，都不应将其安装在由 Poetry 管理的项目环境中**。这可确保 Poetry 自己的依赖项不会被意外升级或卸载。（以下每种安装方法都可确保将 Poetry 安装到隔离的环境中。此外，不应激活安装了诗歌的隔离虚拟环境来运行诗歌命令。
```bash
# pipx安装
pipx install poetry
pipx list

# 升级
pipx upgrade poetry
poetry self update
```
## Project setup
* [Setting up a new project](https://python-poetry.org/docs/basic-usage/#project-setup)
```bash
# 创建新的poetry env poetry-demo
# 默认情况下，Poetry 将尝试使用 Poetry 安装期间使用的 Python 版本为当前项目创建虚拟环境。
poetry new poetry-demo
# 如果你想用与文件夹不同的方式命名你的项目，你可以传递 --name 选项：
poetry new my-folder --name my-package

# poetry全局设置为使用当前激活的env为intepreter
poetry config virtualenvs.prefer-active-python true
poetry config --list

# install 命令从当前项目中读取 pyproject.toml 文件，解析依赖项并安装它们。
# 如果当前目录中有 poetry.lock 文件，它将使用那里的确切版本，而不是解析它们。
poetry install
# 如果您只想安装依赖项，请运行带有 --no-root 标志的 install 命令
poetry install --no-root

# 查看env list
poetry env list
poetry env info

# 删除env
poetry env remove poetry-demo-u3sLpZsf-py3.11

# 安装包
poetry add pandas
poetry add requests@2.12.1
poetry add requests~2.12.1
poetry add requests^2.12.1 

# 进入和退出poetry shell
poetry shell
exit

# 如果要同步环境 – 并确保它与锁定文件匹配 – 请使用 --sync 选项。
poetry install --sync

# 为了获取最新版本的依赖项并更新 poetry.lock 文件，您应该使用 update 命令。
poetry update
poetry update requests toml

# remove 命令从当前已安装的软件包列表中删除软件包。
poetry remove pendulum
poetry remove mkdocs --group docs

# 要列出所有可用的软件包，您可以使用 show 命令。
poetry show
poetry show --tree
poetry show pendulum
poetry show pandas --tree

# check 命令验证 pyproject.toml 文件的内容及其与 poetry.lock 文件的一致性。如果有任何错误，它将返回一份详细的报告。
poetry check
```
* pyproject.toml内容解释
  + `python = "^3.11"`
    - ^3.11 表示可以接受 3.11 及以上版本，但必须是 3.x 系列，且不能超过 4.0。例如，3.11.0、3.11.1、3.12 等都是符合这个约束的版本，但 4.0.0 则不符合
  + `package-mode = false`
    - 如果你只想将 Poetry 用于依赖项管理而不用于打包，你可以使用非 package 模式：
  + `poetry version minor`
    - 例如把：version = "0.1.0"升级一个小版本到：version = "0.2.0"  
### Initialising a pre-existing project
```bash
cd pre-existing-project
poetry init
```
### 将依赖项更新到最新版本
* 如上所述，poetry.lock 文件会阻止您自动获取最新版本的依赖项。要更新到最新版本，请使用 update 命令。这将获取最新的匹配版本（根据您的 pyproject.toml 文件）并使用新版本更新锁定文件。（这相当于删除 poetry.lock 文件并再次运行 install。
  + 如果 poetry.lock 和 pyproject.toml 未同步，Poetry 将在执行安装命令时显示 Warning。
  ```bash
  poetry update
  poetry update pandas
  ```
## 管理依赖项
### Dependency groups  依赖项组
* Poetry 提供了一种按组组织依赖项的方法。例如，您可能拥有仅用于测试项目或构建文档的依赖项。
* 要声明新的依赖项组，请使用 tool.poetry.group.<group> 部分，其中 <group> 是依赖项组的名称（例如，test）：
```toml
[tool.poetry.group.test]  # This part can be left out

[tool.poetry.group.test.dependencies]
pytest = "^6.0.0"
pytest-mock = "*"
```
* 所有依赖项都**必须跨组彼此兼容**，因为无论是否需要安装，它们都会被解析（请参阅安装组依赖项）。
#### Optional groups  可选组
* 依赖项组可以声明为可选。当您有一组仅在特定环境或特定用途中需要的依赖项时，这是有意义的。
```toml
[tool.poetry.group.docs]
optional = true

[tool.poetry.group.docs.dependencies]
mkdocs = "*"
```
* 除了默认依赖项之外，还可以使用 install 命令的 --with 选项安装可选组：`poetry install --with docs`
* **可选的组依赖项仍将与其他依赖项一起解析，因此应特别注意确保它们彼此兼容。**
#### 向组添加依赖项
* add 命令是将依赖项添加到组的首选方法。这是通过使用 --group （-G） 选项完成的。如果该组尚不存在，则将自动创建该组。`poetry add pytest --group test`
#### 安装组依赖项
* 默认情况下，在执行 poetry install 时，将安装所有非可选组`optional = false`中的依赖项。 
  + 项目的默认依赖项集包括 tool.poetry.dependencies 中定义的隐式主组，以及未明确标记为可选组的所有组。
* 您可以使用 --without 选项排除一个或多个组：`poetry install --without test,docs`
* 您还可以使用 --with 选项选择加入可选组：`poetry install --with docs`
* 一起使用时，**--without 优先于 --with**。例如，以下命令将仅安装可选测试组中指定的依赖项。`poetry install --with test,docs --without docs`
* 在某些情况下，您可能希望仅安装特定的依赖项组，而不安装默认的依赖项集。为此，您可以使用 --only 选项。`poetry install --only docs`
  + 如果只想安装项目的运行时依赖项，则可以使用 --only main 表示法来实现：`poetry install --only main`
  + 如果要安装项目根目录，而不安装其他依赖项，则可以使用 --only-root 选项。`poetry install --only-root`
#### 从组中删除依赖项
* remove 命令支持 --group 选项，用于从特定组中删除包：`poetry remove mkdocs --group docs`
### 同步依赖项
* Poetry 支持所谓的依赖关系同步。依赖项同步可确保 poetry.lock 文件中锁定的依赖项是环境中唯一存在的依赖项，从而删除任何不必要的内容。这是通过使用 install 命令的 --sync 选项来完成的：`poetry install --sync`
  + --sync 选项可以与任何依赖项组相关选项结合使用，以将环境与特定组同步。**请注意，extras 是分开的。任何未选择安装的 extras 都会被删除，无论 --sync 如何。**
  ```bash
  poetry install --without dev --sync
  poetry install --with docs --sync
  poetry install --only dev
  ```
### Layering optional groups  分层可选组
* 省略 --sync 选项时，可以安装可选组的任何子集，而无需删除已安装的可选组。这非常有用，例如，在多阶段 Docker 构建中，您可以在不同的构建阶段多次运行 poetry install。
