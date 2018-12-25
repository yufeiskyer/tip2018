## 再谈数组、集合、字典与 hash、isEqual 方法的关联

**作者**: [halohily](https://weibo.com/halohily)

我们或多或少了解，`Objective-C` 中的 `NSArray`、`NSSet`、`NSDictionary` 与 `NSObject` 及其子类对象的 `hash`、`isEqual` 方法有许多联系，这篇小集讲一下其中的一些细节。

`NSArray` 允许添加重复元素，添加元素时不查重，所以不调用上述两个方法。在移除元素时，会对当前数组内的元素进行遍历，每个元素的 `isEqual` 方法都会被调用（使用 `remove` 方法传入的元素作为参数），所有返回真值的元素都被移除。在字典中，不涉及 `hash` 方法。

NSSet 不允许添加重复元素，所以添加新元素时，该元素的 hash 方法会被调用。若集合中不存在与此元素 `hash` 值相同的元素，则它直接被加入集合，不调用 `isEqual` 方法；若存在，则调用集合内的对应元素的 `isEqual` 方法，返回真值则判等，不加入，处理结束。若返回 `false`，则判定集合内不存在该元素，将其加入。

从集合中移除元素时，首先调用它的 `hash` 方法。若集合中存在与其 `hash` 值相等的元素，则调用该元素的 `isEqual` 方法，若真值则判等，进行移除；若不存在，则会依次调用集合中每个元素的 `isEqual` 方法，只要找到一个返回真值的元素，就进行移除，并结束整个过程。（所以这样会有其他满足 `isEqual` 方法但却被漏掉未被移除的元素）。调用 `contains` 方法时，过程类似。

因此，若某自定义对象会被加入到集合或作为字典的 key 时，需要同时重写 `isEqual` 方法和 `hash` 方法。这样，若集合中某元素存在，则调用它的 `contains` 和 `remove` 方法时，可以在 `O(1)` 完成查询。否则，查询它的时间复杂度提升为 `O(n)`。

值得注意的是，`NSDictionary` 的键和值都是对象类型即可。但是被设为键的对象需要遵守 `NSCopying` 协议。
