# NFT 数字艺术藏品生成器

## 关于

这是一个简单的 Node.js 项目，它使用预先配置的特征和图像层列表来为 NFT 藏品集生成一组独特的图像和元数据文件。 您可以通过更新特征配置和图像层来创建自己的藏品集。

## 入门

### 环境

- [下载并安装 Node.js 和 npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

### 安装

1. 克隆本仓库

2. 在仓库目录中安装`NPM`包

   ```sh
   npm install
   ```

## 使用

`config.js` 文件中有一个示例配置，`traits` 文件夹中还有一些预定义的图像层。 您可以使用该预先存在的配置测试和运行此项目，以首先查看一切如何工作并查看结果。

### 测试

在 `config.js` 文件中测试当前配置

```sh
npm test
```

这只会测试当前配置是否正确。

### 运行

使用当前配置运行项目

```sh
npm run build
```

这将执行主脚本。 如果执行成功，它将：

- 在控制台中打印带有结果统计信息的日志
- 生成一个包含所有令牌图像的文件夹
- 生成一个包含所有令牌元数据文件的文件夹

### 其他脚本

#### 更新图像基础 URI

运行项目后，您可以通过运行以下命令更新所有生成的元数据文件中的图像基本 URI：

```sh
npm run update-base-uri
```

这将获取 `config.js` 中的 `IMAGES_BASE_URI` 的当前值，并使用它来更新所有元数据文件。

#### 创建 GIF

运行项目后，您可以使用生成的图像创建 GIF：

```sh
npm run create-gif
```

#### 计算图像的哈希值

运行项目后，您可以启动以下脚本来计算生成的每个图像的 `SHA-256` 哈希，并打印所有图像的最终出处哈希：

```sh
npm run calculate-hashes
```

## 创建您自己的藏品集

要创建自己的独特令牌藏品集，您只需编辑 `config.js` 文件并更新 `traits` 文件夹中的图像层。

运行这个项目生成的元数据应该兼容[OpenSea 的元数据标准](https://docs.opensea.io/docs/metadata-standards)。 如果您不熟悉这些标准，则应该阅读该页面，因为它有助于理解如何更新 `config.js` 文件。 此外，请确保首先使用示例配置运行项目，并检查生成的元数据文件以进一步了解该过程。

### 修改常量

这些是您需要在 `config.js` 文件中更新的常量：

```JS
config.GIF_FRAMES = 10; // 仅当您想生成GIF
config.IMAGES_BASE_URI = "https://base-uri-to-my-nft-images.com/";
config.IMAGES_HEIGHT = 350;
config.IMAGES_WIDTH = 350;
config.TOKEN_NAME_PREFIX = "My NFT #";
config.TOKEN_DESCRIPTION = "My NFT description.";
config.TOTAL_TOKENS = 100;
```

### 修改特征列表

您还必须修改最后一个名为`ORDERED_TRAITS_LIST`的变量，该变量包含标记的所有可用特征的数组。
每个 _trait_ 都有以下结构：

```JS
{
  display?: string;
  ignore?: boolean;
  type?: string;
  options: {
    allowed?: string[];
    forbidden?: string[];
    image?: string;
    value?: string | number;
    weight: number;
  }[]
}
```

在修改 _traits_ 列表之前，请仔细阅读以下重要说明：

- 对于列表中的每个 _trait_，每个生成的令牌都会从 _options_ 列表中获得**一个**随机选择的 _option_ (_value_ & _image_)。 除非随机选择的 _option_ 结果证明不存在 _value_，在这种情况下，令牌不会从该特定 _trait_ 获得**任何东西**。
- 列表的顺序**很重要！**它将定义图像应该相互合并以创建最终标记图像的顺序。 通常，背景 _trait_ 应该是数组中的第一个。
- _option_ 的随机选择基于其 _weight_ 及其可选的 _allowed/forbidden_ 条件。 _option_ 的 _weight_ 与同一 _options_ 数组中其他项目的 _weights_ **相对**，它应该是至少为 1 的整数。因此，如果将 _weight_ 的 10 放入 _option_ 中，它应该有 同一个数组中 _weight_ 等于 1 的 _option_ 有 10 次以上的机会被选中。
- 如果 _trait_ 被标记为 _ignore_，则在定义令牌唯一性时不会考虑该 _trait_。 例如，如果您不希望标记的背景影响它们的唯一性，那么您可以使用 `ignore: true` 标记该背景 _trait_。
- 可选的 _allowed/forbidden_ 数组应包含一个或多个与前一个 _traits_ 的 _option values_ 匹配的字符串。 使用时，它将使这个 _option_ **only** 允许/禁止具有 **至少** 之前选择的那些字符串 _values_ 之一的令牌。 作为参考，请查看在 `config.js` 文件中用作示例的 _allowed_ 和 _forbidden_ 数组。 在这种情况下，“兰花”三角形仅适用于具有“珊瑚”**或**“薄荷”背景的代币； 并且“蓝绿色”三角形将**不**可用于具有“罗宾”背景的代币。
- _trait_ 中定义的每个 _type_ 都应该是唯一的。
- 如果您留下没有 _type_ 字段的特定 _trait_，它将被视为“通用”_trait_。 重要的是，这些 _traits_ 在它们的 _options_ 数组中与其他 _traits_ 没有任何共同的 _values_。
- 每个 _image_ 字符串都应具有特定 PNG 图像的相对路径。
- 如果您没有在每个 _option_ 中使用定义的 _value_ 放置 _image_ 字段，则您的某些令牌（即使具有唯一的元数据）可能会生成相同的图像。
- _display_ 字段仅用于与数字 _values_ 一起使用。 阅读 [OpenSea 的元数据标准](https://docs.opensea.io/docs/metadata-standards) 了解更多信息。
- 根据您拥有的 _traits_ 数量和它们的 _options_ 数量，您将拥有可以生成的最大数量的唯一令牌。 不建议生成确切的最大可能数量的唯一标记，因为脚本将继续搜索，无论几率如何，直到找到每个组合，使加权因子无用。 作为建议，我会说如果你想生成 N 个令牌，那么创建一个 _traits_ 列表，它可以给你**至少** 2N 个令牌。 当您尝试运行它时，该过程将让您知道“TOTAL_TOKENS”的值是否太大。

> 运行命令 `npm test` 将验证这些规则集是否在您当前的配置中被考虑在内，并且生成的元数据是否符合标准。 用它！
