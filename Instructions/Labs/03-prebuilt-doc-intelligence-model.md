---
lab:
  title: 使用 Azure AI 文档智能预生成模型分析表单
  description: 使用预生成的 Azure AI 文档智能模型处理文档中的文本字段。
---

# 使用 Azure AI 文档智能预生成模型分析表单

在本练习中，你将设置一个 Azure AI Foundry 项目，其中包含文档分析所需的所有必要资源。 你将使用 Azure AI Foundry 门户和 Python SDK 将表单提交到该资源进行分析。

尽管本练习基于 Python，但你也可以使用多种语言特定的 SDK 开发类似的应用程序，包括：

- [适用于 Python 的 Azure AI 文档智能客户端库](https://pypi.org/project/azure-ai-formrecognizer/)
- [适用于 Microsoft .NET 的 Azure AI 文档智能客户端库](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [适用于 JavaScript 的 Azure AI 文档智能客户端库](https://www.npmjs.com/package/@azure/ai-form-recognizer)

此练习大约需要 **30** 分钟。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在浏览器中，浏览到 `https://ai.azure.com/managementCenter/allResources`，并选择“新建”****。 然后选择创建新的 **AI 中心资源**的选项。
1. 在**创建项目**向导中，输入有效的项目名称，并选择创建新中心。 然后，使用**重命名中心**链接为你的新中心指定一个有效名称，展开“**高级选项**”，并为项目配置以下设置：
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - 区域****： *任何可用区域*

    > **注意**：如果你使用的 Azure 订阅启用了用于限制资源名称的策略，可能需要点击“**创建新项目**”对话框底部的链接，转到 Azure 门户以创建该中心。

    > **提示**：如果“**创建**”按钮仍然禁用，请确认你已将中心名称更改为一个唯一的字母和数字组合。

1. 等待创建项目。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 使用读取模型

首先，使用“Azure AI Foundry”门户和读取模型分析具有多种语言的文档：****

1. 在左侧的导航面板中，选择“AI 服务”。****
1. 在“Azure AI 服务”页面中，选择“视觉 + 文档”磁贴。********
1. 在“视觉 + 文档”页面中，确认已选择“文档”选项卡，然后选择“OCR/读取”磁贴。************

    在“读取”页面中，使用你的项目创建的 Azure AI 服务资源应已连接。****

1. 在左侧文档列表中，选择 read-german.pdf****。

    ![显示 Azure AI 文档智能工作室中的“读取”页面的屏幕截图。](./media/read-german-sample.png#lightbox)

1. 在顶部工具栏上，选择“分析选项”，然后在“分析选项”窗格中启用“语言”复选框（位于“可选检测”下）并选择“保存”。******************** 
1. 在左上角，选择**运行分析**。
1. 分析完成后，从图像中提取的文本将显示在**内容**选项卡中的右侧。查看此文本并将其与原始图像中的文本进行比较，以确保准确性。
1. 选择**结果**选项卡。此选项卡显示提取的 JSON 代码。 

## 准备在 Cloud Shell 中开发应用

现在，让我们探索使用 Azure 文档智能服务 SDK 的应用。 你将使用 Cloud Shell 开发你的应用。 你的应用的代码文件已在 GitHub 存储库中提供。

这是代码将分析的发票。

![显示发票文档示例的截图。](./media/sample-invoice.png)

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“终结点和密钥”区域中，选择“Azure AI 服务”选项卡，并记下“API 密钥”和“Azure AI 服务终结点”。**************** 你将使用这些凭据连接到客户端应用程序中你的 Azure AI 服务。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。
1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

    ***按照所选编程语言的步骤操作。***

1. 克隆存储库后，导航到包含代码文件的文件夹：

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令安装将使用的库：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，将 YOUR_ENDPOINT 和 YOUR_KEY 占位符替换为你的 Azure AI 服务终结点及其 API 密钥（从 Azure AI Foundry 门户复制）。********
1. 替换占位符后，使用 Ctrl+S**** 命令保存更改，然后使用 Ctrl+Q**** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

## 添加代码以使用 Azure 文档智能服务

现在你可以使用 SDK 评估 pdf 文件了。

1. 输入以下命令以编辑已提供的应用文件：

    ```
   code document-analysis.py
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，找到注释“导入所需的库”并添加以下代码：****

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. 找到注释“创建客户端”并添加以下代码（注意保持正确的缩进级别）：****

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. 找到注释“分析发票”并添加以下代码：****

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. 找到注释“向用户显示发票信息”并添加以下代码：****

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. 在代码编辑器中，使用 Ctrl+S 命令或通过右键单击 > 单击“保存”以保存更改。******** 使代码编辑器保持打开状态，以防需要修复代码中的任何错误，但应调整窗格大小，以便清楚地看到命令行窗格。

1. 在命令行窗格中，输入以下命令以运行应用程序。

    ```
    python document-analysis.py
    ```

该计划显示具有置信度级别的供应商名称、客户名称和发票总计。 将报告的值与在本部分开头呈现的示例发票进行比较。

## 清理

如果你已使用完 Azure 资源，请记得在 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`) 中删除该资源，以避免产生进一步费用。
