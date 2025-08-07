---
lab:
  title: 开发内容理解客户端应用程序
  description: 使用 Azure AI 内容理解 REST API 为分析器开发客户端应用。
---

# 开发内容理解客户端应用程序

在本练习中，你将使用 Azure AI 内容理解创建从名片中提取信息的分析器。 然后，你将开发使用该分析器从扫描的名片中提取联系人详细信息的客户端应用程序。

此练习大约需要 **30** 分钟。

## Azure AI Foundry 中心和项目

我们将在本练习中使用的 Azure AI Foundry 功能，需要基于 Azure AI Foundry *中心*资源的项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在浏览器中，浏览到 `https://ai.azure.com/managementCenter/allResources`，并选择“新建”****。 然后选择创建新的 **AI 中心资源**的选项。
1. 在**创建项目**向导中，输入有效的项目名称，并选择创建新中心。 然后，使用**重命名中心**链接为你的新中心指定一个有效名称，展开“**高级选项**”，并为项目配置以下设置：
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **中心名称**：中心的有效名称
    - 位置****：选择以下位置之一：\*
        - 澳大利亚东部
        - 瑞典中部
        - 美国西部

    > \*在撰写本文时，Azure AI 内容理解仅在这些区域中可用。

    > **提示**：如果“**创建**”按钮仍然禁用，请确认你已将中心名称更改为一个唯一的字母和数字组合。

1. 等待你的项目创建完成，然后导航到你的项目概述页面。

## 使用 REST API 创建内容理解分析器

你将使用 REST API 创建可从名片图像中提取信息的分析器。

1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

    关闭任何欢迎通知以查看 Azure 门户主页。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **提示**：调整窗格大小，使你尽管主要在 Cloud Shell 中工作，但仍能在Azure 门户页面中看到密钥和终结点 - 你需要将它们复制到你的代码中。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：在 Cloudshell 中时输入命令时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含你的应用代码文件的文件夹：

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    该文件夹包含两张扫描的名片图像以及生成应用所需的 Python 代码文件。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装将要使用的库：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，将 YOUR_ENDPOINT 和 YOUR_KEY 占位符替换为你的 Azure AI 服务终结点及其任一密钥（从 Azure 门户复制），并确保“ANALYZER_NAME”设置为 `business-card-analyzer`。************
1. 替换占位符后，使用 Ctrl+S**** 命令保存更改，然后使用 Ctrl+Q**** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

    > **提示**：现在可以最小化 Cloud Shell 窗格了。

1. 在 Cloud Shell 命令行中，输入以下命令以查看提供的“biz-card.json”JSON 文件：****

    ```
   cat biz-card.json
    ```

    滚动 Cloud Shell 窗格以查看文件中的 JSON，它定义了适用于名片的分析器架构。

1. 查看分析器的 JSON 文件后，输入以下命令以编辑提供的 create-analyzer.py Python 代码文件：****

    ```
   code create-analyzer.py
    ```

    Python 代码文件在代码编辑器中打开。

1. 查看代码，内容包括：
    - 从 biz-card.json 文件加载分析器架构。****
    - 从环境配置文件中检索终结点、密钥和分析器名称。
    - 调用名为 create_analyzer 的函数，该函数当前尚未实现****

1. 在 create_analyzer 函数中，找到注释“创建内容理解分析器”并添加以下代码（注意保持正确的缩进）：********

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. 查看你所添加的代码，该代码将：
    - 为 REST 请求创建适当的标头
    - 提交 HTTP DELETE 请求以删除分析器（如果已存在）。**
    - 提交 HTTP PUT 请求以启动分析器的创建。**
    - 检查响应以检索 Operation-Location 回叫 URL。**
    - 重复向该回叫 URL 提交 HTTP GET 请求以检查操作状态，直至它不再运行。**
    - 向用户确认操作成功（或失败）。

    > **注意**：代码包含一些故意的时间延迟，以避免超出服务的请求速率限制。

1. 使用 Ctrl+S 命令保存代码更改，但应使代码编辑器窗格保持打开状态，以防需要更正代码中的任何错误。**** 调整窗格大小，以便清楚地看到命令行窗格。
1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行 Python 代码：

    ```
   python create-analyzer.py
    ```

1. 查看程序的输出，输出应指示已创建分析器。

## 使用 REST API 分析内容

创建分析器后，可以通过内容理解 REST API 从客户端应用程序使用它。

1. 在 Cloud Shell 命令行中，输入以下命令以编辑提供的 read-card.py Python 代码文件：****

    ```
   code read-card.py
    ```

    Python 代码文件在代码编辑器中打开：

1. 查看代码，内容包括：
    - 识别要分析的图像文件，默认为 biz-card-1.png。****
    - 从项目检索你的 Azure AI 服务资源的终结点和密钥（使用当前 Cloud Shell 会话中的 Azure 凭据进行身份验证）。
    - 调用名为 analyze_card 的函数，该函数当前尚未实现****

1. 在 analyze_card 函数中，找到注释“使用内容理解分析图像”并添加以下代码（注意保持正确的缩进）：********

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. 查看你所添加的代码，该代码将：
    - 读取图像文件的内容
    - 设置要使用的内容理解 REST API 的版本
    - 向你的内容理解终结点提交 HTTP POST 请求，指示分析图像。**
    - 检查该操作的响应以检索分析操作的 ID。
    - 重复向你的内容理解终结点提交 HTTP GET 请求以检查操作状态，直至它不再运行。**
    - 如果操作成功，则保存 JSON 响应，然后分析 JSON 并显示为每个特定类型的字段检索到的值。

    > **注意**：在我们的简单名片架构中，所有字段均为字符串。 此处的代码说明了需要检查每个字段的类型，以便你可以从更复杂的架构中提取不同类型的值。

1. 使用 Ctrl+S 命令保存代码更改，但应使代码编辑器窗格保持打开状态，以防需要更正代码中的任何错误。**** 调整窗格大小，以便清楚地看到命令行窗格。
1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行 Python 代码：

    ```
   python read-card.py biz-card-1.png
    ```

1. 查看程序的输出，该输出应显示以下名片中字段的值：

    ![Adventure Works Cycles 的一名员工 Roberto Tamburello 的名片。](./media/biz-card-1.png)

1. 使用以下命令以不同的名片运行程序：

    ```
   python read-card.py biz-card-2.png
    ```

1. 查看结果，结果应反映此名片中的值：

    ![Contoso 的一名员工 Mary Duartes 的名片。](./media/biz-card-2.png)

1. 在 Cloud Shell 命令行窗格中，使用以下命令查看返回的完整 JSON 响应：

    ```
   cat results.json
    ```

    滚动以查看 JSON。

## 清理

如果已完成内容理解服务的使用，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 在 Azure 门户中，删除在本练习中创建的资源。
