semver语义化版本规范规则和如何校验
---

semver的简单介绍，可以查看以下文章[semver简介](https://www.jianshu.com/p/a7490344044f)


> semver 的使用方法

```typescript
const semver = require('semver')
 
semver.valid('1.2.3') // '1.2.3'
semver.valid('a.b.c') // null
semver.clean('  =v1.2.3   ') // '1.2.3'
semver.satisfies('1.2.3', '1.x || >=2.5.0 || 5.0.0 - 7.2.3') // true
semver.gt('1.2.3', '9.8.7') // false
semver.lt('1.2.3', '9.8.7') // true
semver.minVersion('>=1.0.0') // '1.0.0'
semver.valid(semver.coerce('v2')) // '2.0.0'
semver.valid(semver.coerce('42.6.7.9.3-alpha')) // '42.6.7'
```

以上就是使用的基本方式和方法，主要是比对校验，方便使用

简单说明一下用法

```typescript
// 计较两个版本号的大小
semver.gt('1.2.3', '2.3.4') // false
semver.lt('1.2.3', '2.3.4') // true
```

```typescript
// 验证版本号是否合法，返回null即不合法
semver.valid('1.2.3') // '1.2.3'
semver.valid('a.b.c') // null
```

```text
// 提取版本号
semver.clean('  =v1.2.3   ') // '1.2.3'
semver.major('1.2.3') // '1'
semver.minor('1.2.3') // '2'
semver.patch('1.2.3') // '3'
```

当然还有很多其他方法，

```typescript
// load the whole API at once in a single object
const semver = require('semver')

// or just load the bits you need
// all of them listed here, just pick and choose what you want

// classes
const SemVer = require('semver/classes/semver')
const Comparator = require('semver/classes/comparator')
const Range = require('semver/classes/range')

// functions for working with versions
const semverParse = require('semver/functions/parse')
const semverValid = require('semver/functions/valid')
const semverClean = require('semver/functions/clean')
const semverInc = require('semver/functions/inc')
const semverDiff = require('semver/functions/diff')
const semverMajor = require('semver/functions/major')
const semverMinor = require('semver/functions/minor')
const semverPatch = require('semver/functions/patch')
const semverPrerelease = require('semver/functions/prerelease')
const semverCompare = require('semver/functions/compare')
const semverRcompare = require('semver/functions/rcompare')
const semverCompareLoose = require('semver/functions/compare-loose')
const semverCompareBuild = require('semver/functions/compare-build')
const semverSort = require('semver/functions/sort')
const semverRsort = require('semver/functions/rsort')

// low-level comparators between versions
const semverGt = require('semver/functions/gt')
const semverLt = require('semver/functions/lt')
const semverEq = require('semver/functions/eq')
const semverNeq = require('semver/functions/neq')
const semverGte = require('semver/functions/gte')
const semverLte = require('semver/functions/lte')
const semverCmp = require('semver/functions/cmp')
const semverCoerce = require('semver/functions/coerce')

// working with ranges
const semverSatisfies = require('semver/functions/satisfies')
const semverMaxSatisfying = require('semver/ranges/max-satisfying')
const semverMinSatisfying = require('semver/ranges/min-satisfying')
const semverToComparators = require('semver/ranges/to-comparators')
const semverMinVersion = require('semver/ranges/min-version')
const semverValidRange = require('semver/ranges/valid')
const semverOutside = require('semver/ranges/outside')
const semverGtr = require('semver/ranges/gtr')
const semverLtr = require('semver/ranges/ltr')
const semverIntersects = require('semver/ranges/intersects')
const simplifyRange = require('semver/ranges/simplify')
const rangeSubset = require('semver/ranges/subset')
```

具体还是要看文档，因为比较简单，直接挂载[链接直通车](https://github.com/npm/node-semver)
