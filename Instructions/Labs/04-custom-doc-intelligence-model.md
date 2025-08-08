---
lab:
  title: 使用 Azure AI 文档智能模型分析表单
  description: 创建自定义文档智能模型，以从文档中提取特定数据。
---

# 使用 Azure AI 文档智能模型分析表单

假设公司当前要求员工手动购买订单表并将数据输入到数据库中。 他们希望利用 AI 服务改进数据输入过程。 你决定构建一个机器学习模型，该模型将读取表单并生成可用于自动更新数据库的结构化数据。

**Azure AI** 文档智能是一项 Azure AI 服务，可帮助用户构建自动数据处理软件。 此软件可使用光学字符识别 (OCR) 从表单中提取文字、密钥对和表。 Azure AI 文档智能预置了用于识别发票、收据和名片的模型。 该服务还提供训练自定义模式的功能。 在本练习中，我们将重点关注生成自定义模型。

尽管本练习基于 Python，但你也可以使用多种语言特定的 SDK 开发类似的应用程序，包括：

- [适用于 Python 的 Azure AI 文档智能客户端库](https://pypi.org/project/azure-ai-formrecognizer/)
- [适用于 Microsoft .NET 的 Azure AI 文档智能客户端库](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [适用于 JavaScript 的 Azure AI 文档智能客户端库](https://www.npmjs.com/package/@azure/ai-form-recognizer)

此练习大约需要 **30** 分钟。

## 创建 Azure AI 文档智能资源

要使用 Azure AI 文档智能服务，需要在 Azure 订阅中添加 Azure AI 文档智能或 Azure AI 服务资源。 使用 Azure 门户创建资源。

1. 在一个浏览器标签页中，打开 Azure 门户 (`https://portal.azure.com`)，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。
1. 在 Azure 门户主页上，导航到顶部搜索框并输入**文档智能**，然后按 **Enter**。
1. 在“**文档智能**”页中，选择“**创建**”。
1. 在“创建文档智能”页面上，使用以下设置创建新资源：****
    - **订阅**：Azure 订阅。
    - **资源组**：创建或选择资源组
    - 区域****：任何可用区域
    - **名称**：你的文档智能资源的有效名称
    - **定价层**：免费 F0（如果没有可用的免费层，请选择“标准 S0”）。**
1. 当部署完成后，选择“**转到资源**”，查看资源的“**概览**”页面。

## 准备在 Cloud Shell 中开发应用

你将使用 Cloud Shell 开发你的文本翻译应用。 你的应用的代码文件已在 GitHub 存储库中提供。

1. 在 Azure 门户中，使用页面顶部搜索栏右侧的 **[\>_]** 按钮，在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 调整 Cloud Shell 窗格大小，以便可以同时查看命令行控制台和 Azure 门户。 在两个窗格之间切换时，需要使用拆分栏来进行切换。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## 收集用于训练的文档

您将使用像这样的示例表单来训练和测试模型： 

![本项目中使用的发票图像。](./media/Form_1.jpg)

1. 在命令行中，运行 `ls ./sample-forms` 以列出 sample-forms 文件夹中的内容。**** 注意该文件夹中有以 **.json** 和 **.jpg** 结尾的文件 。

    你将使用 **.jpg** 文件来训练模型。  

    已生成 **.json** 文件，其中包含标签信息。 文件将连同表单一起上传到 blob 存储容器中。

1. 在 Azure 门户中，导航到你的资源的“概述”页面（如果你尚未位于该页面）。******** 在“基本信息”部分下，记下“资源组”、“订阅 ID”和“位置”。************** 在后续步骤中需要使用这些值。
1. 运行命令 `code setup.sh` 以在代码编辑器中打开 setup.sh。**** 你将使用此处理脚本来运行创建需要的其他 Azure 资源所需的 Azure 命令行接口 (CLI) 命令。

1. 在 setup.sh 脚本中，查看命令。**** 程序将会执行以下操作：
    - 在 Azure 资源组中创建存储帐户
    - 将本地 *sampleforms* 文件夹中的文件上传到存储帐户中名为 *sampleforms* 的容器
    - 打印共享访问签名 URI

1. 使用你部署文档智能资源时使用的订阅、资源组和位置名称对应的值，修改 **subscription_id**、**resource_group** 和 **location** 变量声明。

    > **重要说明**：对于你的 location 字符串，请务必使用你所在位置的代码版本。**** 例如，如果你的位置为“美国东部”，则脚本中的字符串应为 `eastus`。 可以看到，该版本是 Azure 门户中你的资源组的“基本信息”选项卡右侧的“JSON 视图”按钮。********

    如果 expiry_date 变量是过去的日期，请将其更新为将来的日期。**** 此变量在生成共享访问签名 (SAS) URI 时使用。 实际操作时，应为 SAS 设置适当的到期日期。 可以在[此处](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works)详细了解 SAS。  

1. 替换占位符后，在代码编辑器中使用 **CTRL+S** 命令或 ** 右键单击 > 保存** 保存更改，然后使用 **CTRL+Q** 命令或 ** 右键单击 > 退出** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

1. 输入以下命令使脚本可执行并运行它：

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. 脚本完成后，查看显示的输出。

1. 在 Azure 门户中，刷新资源组并验证它包含刚刚创建的 Azure 存储帐户。 打开存储帐户，并在左侧窗格中选择“**存储浏览器**”。 然后，在存储浏览器中，展开“Blob 容器”，并选择“sampleforms”容器，以验证文件是否已从本地 custom-doc-intelligence/sample-forms 文件夹上传。************

## 使用文档智能工作室训练模型

现在，你将使用上传到存储帐户的文件来训练模型。

1. 打开新的浏览器选项卡并导航到文档智能工作室 (`https://documentintelligence.ai.azure.com/studio`)。 
1. 向下滚动到**自定义模型**部分，然后选择**自定义提取模型**磁贴。
1. 如果出现提示，请使用你的 Azure 凭据登录。
1. 如果系统询问要使用哪个 Azure AI 文档智能资源，请选择创建 Azure AI 文档智能资源时使用的订阅和资源名称。
1. 在“我的项目”下，使用以下配置创建新项目：****

    - **输入项目详细信息**：
        - 项目名称****：你的项目的有效名称
    - **配置服务资源**：
        - 订阅：Azure 订阅
        - 资源组****：部署你的文档智能资源的资源组
        - **文档智能资源** 你的文档智能资源（选择“设置为默认值”选项并使用默认的 API 版本）**
    - **连接训练数据源**：
        - 订阅：Azure 订阅
        - 资源组****：部署你的文档智能资源的资源组
        - 存储帐户****：由设置脚本创建的存储帐户（选择“设置为默认值”选项，选择 `sampleforms` blob 容器，并将文件夹路径留空）**

1. 创建你的项目后，在页面的右上角，选择“训练”以训练模型。**** 使用以下配置：
    - **模型 ID**：你的模型的有效名称（下一步中需要使用模型 ID 名称）**
    - **构建模式：** 模板。
1. 选择“**转到模型**”。
1. 训练可能需要一些时间。 等到状态显示为“已成功”。****

## 测试自定义文档智能模型

1. 返回到包含 Azure 门户和 Cloud Shell 的浏览器选项卡。 在命令行中，运行以下命令以更改为包含应用程序代码文件的文件夹：

    ```
    cd Python
    ```

1. 通过运行以下命令安装文档智能包：

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

1. 在包含 Azure 门户的窗格中，在你的文档智能资源的“概述”页面上，选择“单击此处管理密钥”以查看资源的终结点和密钥。******** 然后，使用以下值编辑配置文件：
    - 你的文档智能终结点
    - 你的文档智能密钥
    - 训练模型时指定的模型 ID

1. 替换占位符后，使用 Ctrl+S**** 命令保存更改，然后使用 Ctrl+Q**** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

1. 打开你的客户端应用程序的代码文件（C# 为 `code Program.cs`，Python 为 `code test-model.py`），并查看其中包含的代码，尤其是 URL 中的图像是否指向 Web 上此 GitHub 仓库中的文件。 关闭文件，而不进行任何更改。

1. 在命令行中输入以下命令以运行程序：

    ```
   python test-model.py
    ```

1. 查看输出结果，观察模型的输出结果是如何提供字段名称的，如 `Merchant` 和 `CompanyPhoneNumber`。

## 清理

如果您已完成 Azure 资源的使用，请记住在 [Azure 门户](https://portal.azure.com/?azure-portal=true)中删除该资源，以避免进一步收费。

## 详细信息

有关文档智能服务的更多信息，请参阅[文档智能文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true)。
