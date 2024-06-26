`本仓库为2024年全国大学生信息安全竞赛作品赛项目代码仓库（在研）`
# VulnTrac:大语言模型驱动的代码脆弱性检测系统
# System Overview
![PPT学术图-Latest drawio (2)](https://github.com/Nash-Equilibrium/ciscn/assets/90449797/60e10a32-12b4-4392-a35e-a9cd2d18f127)

# 功能介绍
## 1.智能漏洞检测和修复提示
实现路线：
1. 拆解漏洞检测任务为漏洞定位任务、信息检索任务和总结任务。
2. 利用LangChain，通过SimpleSequentialChain将针对子任务所设计的不同Prompt Template进行链式使用。
3. 利用Secure_Programming_DPO数据集进行DPO微调。

## 2.支持对代码仓库实时监控
实现路线：
1. 维护一个Repo缓存数据库，利用request库下载仓库压缩包到本地，利用Celery库保证数据更新，间隔为设定值。
2. Repo缓存数据库中存放仓库压缩包Hash值，将更新后的压缩包Hash与前一项比较，若不同，则执行检测流程。
3. 检测结果形成PDF，通过邮件通知用户。

## 3.支持长文件分析
实现路线：
1. 首先判断代码长度，若超长则利用tree-sitter提取代码的AST。
2. 将提取的AST与LLM交互，自动进行代码注释标注。
3. 将包含代码注释的代码送入LangChain，进行下一步处理。

## 4.支持多源文件分析
实现路线：
1. 如果多个源文件中有长文件，先对其执行长文件的处理。
2. 维护一个过滤名单，以后缀名为匹配字段，将无关文件（例如配置文件、图片等）过滤。
3. 分别对预处理后的文件进行相似度分析，并按相似度降序排列，以达到设置的窗口大小为目标贪心组合代码块。

# 启动方法
## 1.安装依赖
```shell
pip install -r requirements.txt
```
## 2.下载模型
安装ModelScope
```shell
#安装ModelScope
pip install modelscope
```
利用ModelScope SDK来下载:
```python
#SDK模型下载
from modelscope import snapshot_download
model_dir = snapshot_download('No1res/CodeQwen_VulnTracFinetuned')
```
请注意，将模型保存到本地时，需要调整VulnTracCore中的模型路径。

## 3.安装tree-sitter依赖
```bash
bash tree_sitter_setup.sh
```
如果出现问题，请确保您的工作路径为`/VulnTrac/VulnTrac`

## 4.配置Platform
```shell
cd VulnTrac/VulnTracPlatform
npm install
```
## 5.启动后端
启动服务之前，请确保已经安装`flask`与`redis`。确认安装后，启动：
```shell
celery -A app.celery worker --beat --scheduler redis --loglevel=info
```
## 6.启动前端
```shell
cd VulnTrac/VulnTracPlatform
npm run dev
```