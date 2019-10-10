# Contentos

## 概述

Contentos (aka. coschain) 是一个侧重于内容创作的区块链架构，用户通过发表内容，评论或者点赞，投票以参与到 coschain 生态中。在 coschain 中，创作是受到鼓励的：用户可以通过发表内容或者评论获取收益，而用户发表的内容获取到权重越大，那么分得的奖励也会更大。

## COS/VEST/Chicken

权重是对作品获取到的点赞和投票总和的衡量。因此，一旦有用户对作品点赞或者投票，作品的权重也会相应改变。截止目前，投票功能尚未开放，改变权重的方式仅有点赞，而点赞的效力和用户的 VEST 正相关。

coschain 中，基本的流通货币被称为 COS，它的精确位为 6 位小数。COS 在系统内可能有三种形态，分别被称为 COS, VEST 和 Chicken。COS 可以自由流通，由一个账户转移到另一个账户。与之相反，VEST 和 Chicken 都不可以转移。

VEST 是冻结后的 COS，它代表了用户在系统内的能量：如同上文所述，越多的 VEST 也就能在单次点赞中给予作品更多的权重，让作品获得更多奖励；另外，如果需要竞选出块节点（aka. BP 节点），那么充足的 VEST 也是必要的。COS 可以自由地，立即地转化为 VEST，反之由 VEST 转化为 COS 并不自由，并且需要长达 13 周的时间来转化。

Chicken 也是由 COS 转化而来，目的是获得耐力。在 coschain 中，虽然执行交易并不会收取燃料费，但是也会扣除一部分耐力值以防止滥用。用户每天都会有一部分免费的耐力，足够一般用户日常所需。但是如果对执行交易有更大的需求，比如需要执行大量合约，那么就需要质押 COS 为 Chicken 去获取额外的耐力。同样，COS 可以自由，立即转化为 Chicken，反之需要至少三天才能由 Chicken 转为 COS。