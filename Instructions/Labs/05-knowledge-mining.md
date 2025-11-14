---
lab:
  title: 创建知识挖掘解决方案
  description: 使用 Azure AI 搜索从文档中提取关键信息，使其更易于搜索和分析。
---

# 创建知识挖掘解决方案

在本练习中，你将使用 AI 搜索为 Margie's Travel（一家虚构的旅行社）维护的一组文档编制索引。 编制索引过程涉及使用 AI 技能提取关键信息，使其可搜索，并生成包含数据资产的知识存储以供进一步分析。

尽管本练习基于 Python，但你也可以使用多种语言特定的 SDK 开发类似的应用程序，包括：

- [适用于 Python 的 Azure AI 搜索客户端库](https://pypi.org/project/azure-search-documents/)
- [适用于 Microsoft .NET 的 Azure AI 搜索客户端库](https://www.nuget.org/packages/Azure.Search.Documents)
- [适用于 JavaScript 的 Azure AI 搜索客户端库](https://www.npmjs.com/package/@azure/search-documents)

此练习大约需要 **40** 分钟。

## 创建 Azure 资源

你将为 Margie's Travel 创建的解决方案需要使用 Azure 订阅中的多个资源。 在本练习中，你将直接在 Azure 门户中创建它们。 你也可以使用脚本或 ARM 或 BICEP 模板创建它们；或者，可以创建包含 Azure AI 搜索资源的 Azure AI Foundry 项目。

> **重要说明**：应在同一位置中创建你的 Azure 资源！

### 创建 Azure AI 搜索资源

1. 在 Web 浏览器中，打开 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，然后使用你的 Azure 凭据登录。
1. 选择“&#65291;创建资源”按钮，搜索 `Azure AI Search`，并使用以下设置创建“Azure AI 搜索”资源：********
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **服务名称**：你的搜索资源的有效名称**
    - 位置****：任何可用位置**
    - **定价层**：免费

1. 等待部署完成，然后转到部署的资源。
1. 在 Azure 门户的 Azure AI 搜索资源边栏选项卡上查看“概述”**** 页。 在这里，你可以使用可视化界面来创建、测试、管理和监视搜索解决方案的各个组件，包括数据资源、索引、索引器和技能组。

### 创建存储帐户

1. 返回到主页，然后使用以下设置创建“存储帐户”资源：****
    - **订阅**：Azure 订阅
    - 资源组****：你的 Azure AI 搜索和 Azure AI 服务资源所在的同一资源组**
    - **存储帐户名称**：你的存储资源的有效名称**
    - 区域****：与 Azure AI 搜索资源相同的区域**
    - **主服务**：Azure Blob 存储或 Azure Data Lake Storage Gen 2
    - **性能**：标准
    - **冗余**：本地冗余存储 (LRS)

1. 等待部署完成，然后转到部署的资源。

    > **提示**：使存储帐户门户页面保持打开状态 - 你将在下一过程中使用它。

## 将文档上传到 Azure 存储

你的知识挖掘解决方案将从 Azure 存储 Blob 容器中的旅行手册文档中提取信息。

1. 在新的浏览器选项卡中，从 `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` 下载 [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip)，并将其保存到本地文件夹中。
1. 解压缩下载的 documents.zip 文件，并查看它包含的旅行手册文件。** 你将从这些文件中提取信息并为其编制索引。
1. 在包含你的存储帐户的 Azure 门户页面的浏览器选项卡中，在左侧的导航窗格中，选择“存储浏览器”。****
1. 在存储浏览器中，选择“Blob 容器”****。

    目前，你的存储帐户应仅包含默认的“$logs”容器。****

1. 在工具栏中，选择“+ 容器”并使用以下设置创建新容器：****
    - **名称**：`documents`
    - **匿名访问级别**：专用(不允许匿名访问)\*

    > **注意**：\*除非在创建你的存储帐户时启用了允许匿名容器访问的选项，否则将无法选择任何其他设置！

1. 选择“documents”容器将其打开，然后使用“上传”工具栏按钮将之前从 documents.zip 解压缩的 .pdf 文件上传到容器的根中，如下所示：************

    ![Azure 存储浏览器的屏幕截图，其中包含 documents 容器及其文件内容。](./media/blob-containers.png)

## 创建并运行索引器

现在你已准备好文档了，接下来可以创建索引器从中提取信息。

1. 在 Azure 门户中，浏览到 Azure AI 搜索资源。 然后，在其“概述”页上，选择“导入数据” 。

    ![Azure 搜索服务的屏幕截图，其中突出显示了“导入数据”。](./media/overview-panel.png)
    
    > **注意：** 在 Azure AI 搜索资源的“概述”**** 页面中，工具栏提供了两个选项：  
    > - **导入数据**（经典体验）  
    > - **导入数据（新）**（新体验）  
    >  
    > 此外，“概述”部分下的“开始”面板中包含一个“导入”按钮，它会将您重定向到新 UI************。  
    >  
    > 本课程中的说明参考的是经典导入数据工作流****。 为避免混淆，请确保从工具栏中选择“导入数据”****。
1. 在“连接到数据”页面上的“数据源”列表中，选择“Azure Blob 存储”  。 然后使用以下值补全数据存储详细信息：
    - **数据源**：Azure Blob 存储
    - **数据源名称**：`margies-documents`
    - **要提取的数据**：内容和元数据
    - **分析模式**：默认
    - **订阅**：Azure 订阅
    - **连接字符串：** 
        - 选择“选择现有连接”****
        - 选择存储帐户
        - 选择“documents”容器****
    - **托管标识身份验证**：无
    - **容器名称**：documents
    - **Blob 文件夹**：将此项留空
    - **说明**：`Travel brochures`
1. 继续下一步（添加认知技能），其中包含三个可扩展的部分需要完成。****
1. 在“附加 Azure AI 服务”部分中，选择“免费(有限扩充)”\*。********

    > **注意**：\*用于 Azure AI 搜索的免费 Azure AI 服务资源可用于最多为 20 个文档编制索引。 在实际解决方案中，应在订阅中创建 Azure AI 服务资源，以便为更多文档启用 AI 扩充。

1. 在“添加扩充”部分中：
    - 将“技能组名称”更改为 `margies-skillset`。****
    - 选择“启用 OCR”选项，将所有文本合并到 merged_content 字段中。
    - 确保“源数据”字段设置为“merged_content” 。
    - 将“扩充信息粒度级别”保留为“源字段”，即将文档的全部内容设为可编制索引；但请注意，可将此项更改为在更精细的级别（例如页面或语句）提取信息。
    - 选择以下经扩充的字段：

        | 认知技能 | 参数 | 字段名称 |
        | --------------- | ---------- | ---------- |
        | **文本认知技能** | |  |
        | 提取人名 | | 用户 |
        | 提取位置名称 | | locations |
        | 提取关键短语 | | keyphrases |
        | **图像认知技能** | |  |
        | 由图像生成标记 | | imageTags |
        | 从映像中生成描述文字 | | imageCaption |

        再次检测你的选择（稍后可能很难更改这些选项）。

1. 在“将扩充保存到知识存储”部分中：****
    - 仅选中以下复选框（将显示错误，你稍后即可解决）：<font color="red"></font>
        - **Azure 文件投影**：
            - 图像投影
        - **Azure 表投影**：
            - 文档
                - 关键短语
        - **Azure blob 投影**：
            - 文档
    - 在“存储帐户连接字符串”下（“错误消息”下方）：****<font color="red"></font>
        - 选择“选择现有连接”****
        - 选择存储帐户
        - 选择“documents”容器（这仅在浏览界面中选择存储帐户时才需要 - 你将为提取的知识资产指定不同的容器名称！）******
    - 将“容器名称”更改为 `knowledge-store`。****
1. 继续下一步（自定义目标索引），你将在其中指定索引的字段。**** 
1. 将“索引名称”更改为 `margies-index`。****
1. 确保“密钥”设置为“metadata_storage_path”，将“建议器名称”留空，并确保“搜索模式”为“analyzingInfixMatching”。********************
1. 对索引字段进行以下更改，并让其他所有字段保留其默认值（重要提示：可能需要向右滚动才能看到整张表）：

    | 字段名称 | 可检索 | 可筛选 | 可排序 | 可查找 | 可搜索 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 用户 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    再次检查你的选择，特别注意确保已为每个字段选择了正确的“可检索”、“可筛选”、“可排序”、“可分面”和“可搜索”选项（稍后可能很难更改这些选项）。********************

1. 继续下一步（创建索引器），你将在其中创建并计划索引器。****
1. 将“索引器名称”更改为 `margies-indexer`。****
1. 将“计划”设置为“一次”。
1. 选择“提交”以创建数据源、技能组、索引和索引器。 索引器将自动运行并运行索引管道，该索引管道可以：
    - 从数据源中提取文档元数据字段和内容
    - 运行认知技能的技能组，以生成更多的扩充字段
    - 将提取的字段映射到索引。
    - 将提取的数据资产保存到知识存储。
1. 在左侧的导航窗格中，在“搜索管理”下，查看“索引器”页面，该页面应显示新创建的“margies-indexer”。************ 等待几分钟，然后单击“&#8635; 刷新”，直到“状态”指示“成功”************。

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 返回到你的 Azure AI 搜索资源的“概述”页面，在工具栏上，选择“搜索资源管理器”。********
1. 在搜索资源管理器中的“查询字符串”框中，输入 `*`（单个星号），然后选择“搜索” 。

    此查询将以 JSON 格式检索索引中的所有文档。 检查结果并记下每个文档的字段，其中包含由你选择的认知技能所提取的文档内容、元数据和经扩充的数据。

1. 在“视图”**** 菜单中，选择“JSON 视图”**** 并注意已显示搜索的 JSON 请求，如下所示：

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. 结果包含“@odata.count”字段，它显示在结果顶部，指示搜索返回的文档数。****

1. 修改 JSON 请求以包含 select 参数，如下所示：****

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    这次，结果仅包含文件名和文档内容中提及的任何位置。 文件名位于从源文档提取的“metadata_storage_name”字段中。**** “locations”字段是由 AI 技能生成的。****

1. 现在尝试以下查询字符串：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    此搜索会查找任意可搜索字段中提及“New York”的文档，并返回文件名和文档中的关键短语。

1. 让我们尝试再运行一个查询：

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    此查询返回任何提及“New York”且大小小于 380,000 字节的文档的文件名和关键短语。

## 创建搜索客户端应用程序

现在你拥有了有用的索引，可从客户端应用程序使用它。 可通过下列方式进行此操作：使用 REST 接口，通过 HTTP 以 JSON 提交请求并收取响应；使用你偏好的编程语言的软件开发工具包 (SDK)。 在此练习中，我们将使用 SDK。

> **注意**：可选择将该 SDK 用于 C# 或 Python 。 在下面的步骤中，请执行适用于你的语言首选项的操作。

### 获取搜索资源的终结点和密钥

1. 在 Azure 门户中，关闭“搜索资源管理器”页面，并返回到你的 Azure AI 搜索资源的“概述”页面。****

    请注意 URL 值，该值应类似于 **https://*your_resource_name*.search.windows.net**。**** 这是你的搜索资源的终结点。

1. 在左侧的导航窗格中，展开“设置”并查看“密钥”页面。********

    请注意，有两个“管理员”密钥和一个“查询”密钥。******** 管理密钥用于创建和管理搜索资源；查询密钥由只需要执行搜索查询的客户端应用程序使用。

    你需要你的客户端应用程序的终结点和“查询”密钥。******

### 准备使用 Azure AI 搜索 SDK

1. 使用 Azure 门户顶部搜索栏右侧的“[\>_]”按钮在 Azure 门户中创建新的 Cloud Shell，选择订阅中不含存储的 PowerShell 环境。**********

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。 最初，你需要同时查看 Cloud Shell 和 Azure 门户（以便可以找到并复制所需的终结点和密钥）。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：在 Cloudshell 中时输入命令时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. 通过运行以下命令安装 Azure AI 搜索 SDK 和 Azure 标识包：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. 运行以下命令来编辑你的应用的配置文件：

    ```
   code .env
    ```

    配置文件在代码编辑器中打开。

1. 编辑配置文件以替换以下占位符值：

    - **your_search_endpoint**（替换为你的 Azure AI 搜索资源的终结点）**
    - **your_query_key**（替换为你的 Azure AI 搜索资源的查询密钥）**
    - **your_index_name**（替换为你的索引名称，应为 `margies-index`）**

1. 更新占位符后，使用 Ctrl+S 命令保存文件，然后使用 Ctrl+Q 命令将其关闭。********

    > **提示**：现在你已从 Azure 门户复制了终结点和密钥，接下来你可能想将 Cloud Shell 窗格最大化，以便更轻松地使用。

1. 运行以下命令以打开你的应用的代码文件：

    ```
   code search-app.py
    ```

    代码文件在代码编辑器中打开。

1. 查看代码，注意它执行以下操作：

    - 从你编辑的配置文件中检索你的 Azure AI 搜索资源和索引的配置设置。
    - 使用终结点、密钥和索引名称创建 SearchClient 以连接到你的搜索服务。****
    - 提示用户输入搜索查询（直到他们输入“退出”）
    - 使用查询搜索索引，返回以下字段（按metadata_storage_name 排序）：
        - metadata_storage_name
        - locations
        - 用户
        - keyphrases
    - 分析返回的搜索结果以显示结果集中为每个文档返回的字段。

1. 关闭代码编辑器窗格 (CTRL+Q)，使 Cloud Shell 命令行控制台窗格保持打开状态**
1. 输入以下命令以运行应用：

    ```
   python search-app.py
    ```

1. 出现提示时，输入查询，例如 `London` 并查看结果。
1. 尝试另一个查询，例如 `flights`。
1. 完成应用测试后，输入 `quit` 将其关闭。
1. 关闭 Cloud Shell，返回到 Azure 门户。

## 查看知识存储

运行使用技能组创建知识存储的索引器后，由索引过程提取的扩充数据将持久保存在知识存储投影中。

### 查看对象投影

对于每个索引文档，Margie's Travel 技能组中定义的对象投影由 JSON 文件组成。 这些文件存储在技能组定义中指定的 Azure 存储帐户中的 Blob 容器中。

1. 在 Azure 门户中，查看之前创建的 Azure 存储帐户。
1. （在左侧窗格中）选择“存储浏览器”选项卡，在 Azure 门户的存储资源管理器界面中查看存储帐户。
1. 展开“Blob 容器”，查看存储帐户中的容器。 除了存储源数据的“documents”容器外，还应有两个新的容器：“knowledge-store”和“margies-skillset-image-projection”。************ 这些容器都是由索引过程创建的。
1. 选择“knowledge-store”容器。 对于每个索引文档，它都应包含一个文件夹。
1. 打开任意文件夹，然后选择它所包含的 objectprojection.json 文件，并使用工具栏上的“下载”按钮下载并打开它。******** 每个 JSON 文件都包含一个索引文档的表示形式，包括由技能组提取的扩充数据，如下所示（已格式化，使其更易于阅读）。

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

创建此类对象投影的功能使你能够生成可合并到企业数据分析解决方案中的扩充数据对象。**

### 查看文件投影

在索引过程中，技能组中定义的文件投影会为从文档中提取的每个图像都创建 JPEG 文件。

1. 在 Azure 门户的“存储浏览器”界面中，选择“margies-skillset-image-projection”blob 容器。****** 该容器会为每个包含图像的文档包含一个文件夹。
2. 打开任意文件夹并查看其内容，每个文件夹至少包含一个 \*.jpg 文件。
3. 打开任意图像文件，然后下载并查看它以查看图像。

凭借可生成这类文件投影的功能，索引成为了从大量文档中提取嵌入图像的有效方法。

### 查看表投影

技能组中定义的表投影构成了扩充数据的关系架构。

1. 在 Azure 门户的“存储浏览器界面”中，展开“表”。******
2. 选择“margiesSkillsetDocument”表以查看数据。**** 此表包含已编制索引的每个文档的行：
3. 查看“margiesSkillsetKeyPhrases”表，其中包含从文档中提取的每个关键短语的行。****

创建表投影的功能使你能够生成用于查询关系架构的分析和报告解决方案。** 自动生成的键列可用于联接查询中的表 - 例如返回从特定文档中提取的所有关键短语。

## 清理

现已完成练习，请删除所有不再需要的资源。 删除 Azure 资源：

1. 在 **Azure 门户**中，选择“资源组”。
1. 选择不需要的资源组，然后选择“删除资源组”。****

## 详细信息

要详细了解 Azure AI 搜索，请参阅 [Azure AI 搜索文档](https://docs.microsoft.com/azure/search/search-what-is-azure-search)。
