# 布隆过滤器
> 用于准确快速判断某个数据是否在大量数据集合中, 并且不占用内存.

## 简介
### 结构
布隆过滤器是一种数据结构, 由一串很长的二进制向量组成(二进制数组), 初始默认值都为0, 可以将其看作一个容器.  

### 添加数据
要添加的元素为 key, 通过多个 hash 函数, 计算多个 key 对应的 hash 值, 以这个 hash 值作为 index, 将对应数组的 index 位置的元素置 1.  

> 注意数组从 0 开始计数.  

### 判断数据是否存在
同样的, 将新数据通过之前定义的多个 hash 函数, 计算对应的 hash 值作为 index 值, 并在布隆过滤器中判断对应 index 的元素是否都为 1. **如果不都为 1, 则该数据一定不存在.**  
**由于不同数据的 hash 值可能相同, 如果对应 index 的元素都为 1, 不一定表示该数据存在.**  

### 优缺点
- 优点: 使用二进制数组, 占用内存极少, 插入和查询速度都很快.
- 缺点: 随着数组增加, 误判率会增加; 无法判断数据是否一定存在; 无法删除数据.

### 参数设置
主要三个参数:
1. 哈希函数个数 k
2. 布隆过滤器位数组容量 m
3. 布隆过滤器插入数据数量 n

最优准确性条件: $k = ln2 * (m / n)$

## Redis 实现布隆过滤器
> 使用 spring-boot-starter-data-redis + guava

### 下载布隆过滤器  
```bash
git clone https://github.com/RedisLabsModules/redisbloom.git
cd redisbloom
make # 编译redisbloom
```
编译正常执行完，会在根目录生成一个 redisbloom.so 文件  

### 启动 Redis 服务器  
```bash
./src/redis-server redis.conf --loadmodule ./src/modules/RedisBloom-master/redisbloom.so
```

### LUA 控制 BloomFilter
```java
@Component
public class BloomFilterHelper {

    private final StringRedisTemplate redisTemplate;

    @Autowired
    public BloomFilterHelper(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public Object addsLuaBloomFilter(String filterName, List<String> values) {
        DefaultRedisScript<Boolean> bloomAdd = new DefaultRedisScript<>();
        bloomAdd.setScriptSource(new ResourceScriptSource(new ClassPathResource("bloom-filter-insert.lua")));
        bloomAdd.setResultType(Boolean.class);
        return redisTemplate.execute(bloomAdd, values, filterName);
    }

    public Boolean existsLuaBloomFilter(String filterName, String value) {
        DefaultRedisScript<Boolean> bloomExists= new DefaultRedisScript<>();
        bloomExists.setScriptSource(new ResourceScriptSource(new ClassPathResource("bloom-filter-exist.lua")));
        bloomExists.setResultType(Boolean.class);
        List<String> keyList= new ArrayList<>();
        keyList.add(filterName);
        keyList.add(value);
        return redisTemplate.execute(bloomExists,keyList);
    }
}
```
LUA 脚本:
```lua
-- 判断是否存在
local bloomName = KEYS[1]
local value = KEYS[2]
-- BloomFilter
local result_1 = redis.call('BF.EXISTS', bloomName, value)
return result_1
```
```lua
-- 添加元素
local values = KEYS
local bloomName = ARGV[1]
local result_1
for k,v in ipairs(values) do
 result_1 = redis.call('BF.ADD', bloomName, v)
end
return result_1
```