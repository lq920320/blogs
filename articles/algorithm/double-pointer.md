## 解题算法之——双指针

引言：什么是双指针呢？故名思义，就是两个指针。这么说可能有点欠揍，通过问题来说明应该更直观。众所周知，链表有一种数据结构为环形链表，即链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。那么给你一个链表，你怎么判断其中有环呢？其中一种思路是，选定两个指针，同时从比如 `a` 节点出发，一个指针一次走一步，`a.nex`，一个指针一次走两步，`a.next.next`，如果它俩某一次又重合到了同一节点，就说明链表中是有环的。这样对于双指针你就应该有了更直观的认识。

上面提到的双指针，又被称作“快慢指针”，另外，双指针还有“对撞指针”，即从链表或者数组两端往中间走。包括“滑动窗口”也是双指针的思想。今天我们就来看看“双指针”的应用。

### 1、快慢指针

就从引言中的问题来看，如何判断一个列表是环形链表呢？首先给出列表节点的数据结构：

```java
public class ListNode {
  int val;
  ListNode next;
  
  ListNode(int x) {
    val = x;
    next = null;
  }
}
```

节点的数据结构包含本节点的值，以及 `next` 节点。按照快慢指针的思路，我们就可以写出下面的判断代码：

```java
    /**
     * 判断链表是否有环。使用快慢指针即可解决
     *
     * @param head 链表
     * @return 是否有环
     */
    public boolean hasCycle(ListNode head) {
      // 快慢指针都是从头部节点出发
      ListNode fast = head;
      ListNode low = head;
      while (fast != null && fast.next != null) {
        // 快指针一次走两步
        fast = fast.next.next;
        // 慢节点一次走一步
        low = low.next;
        // 当两个指针走到同一位置时，表明链表中存在环形结构，即环形链表
        if (fast == low) {
          return true;
        }
      }
      return false;
    }
```

### 2、对撞指针

所谓对撞指针，就是说我们可以将指向最左侧的索引定义为`左指针(left)`，最右侧的定义为`右指针(right)`，两个指针从两端往中间靠拢，最终到达相同位置，即撞在一起。这种方法一般适用于有序列表或者数组，可以更好地确定位置。**二分查找**也是可以用这种思路来解决的，首先确定中间位置的值，如果比目标值小，则左指针从中间位置向右移，否则，右指针从中间位置向左移。可以简单看下代码：

```java
		/**
     * 二分查找非递归写法
     *
     * @param nums 数组
     * @param target 目标值
     * @param left 左指针
     * @param right 右指针
     * @return 目标值位置
     */
    public static int binarySearch(int[] nums, int target, int left, int right) {
        // 这里需要注意，循环条件
        while (left <= right) {
            // 这里需要注意，计算mid
            int mid = left + ((right - left) >> 1);
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                // 左指针向右移动
                left = mid + 1;
            } else if (nums[mid] > target) {
                // 右指针向左移动
                right = mid - 1;
            }
        }
        // 没有找到该元素，返回 -1
        return -1;
    }
```

LeetCode 上有一个比较典型的问题，救生艇问题：

```
#881. 救生艇
第 i 个人的体重为 people[i]，每艘船可以承载的最大重量为 limit。
每艘船最多可同时载两人，但条件是这些人的重量之和最多为 limit。
返回载到每一个人所需的最小船数。(保证每个人都能被船载)。

示例 1：
输入：people = [1,2], limit = 3
输出：1
解释：1 艘船载 (1, 2)

示例 2：
输入：people = [3,2,2,1], limit = 3
输出：3
解释：3 艘船分别载 (1, 2), (2) 和 (3)

示例 3：
输入：people = [3,5,3,4], limit = 5
输出：4
解释：4 艘船分别载 (3), (3), (4), (5)
```

这时候我们就可以有这么一种思路，既然是求所需的救生艇的最小数量，那么就需要每条船上承载更多重量（数量上已经限定了最多两人）。所以，首先是对重量进行排序，如果最重的人可以与最轻的人共用一艘船，那么就这样安排；否则，最重的人无法与任何人配对，那么他们将自己独自乘一艘船。然后，最轻的人继续检查是否和次重的共用一条船，如果可以，第二轻的人继续和第三重的人进行检查是否可以共用，以此类推……代码如下：

```java
    /**
     * 思路：双指针
     * 如果最重的人可以与最轻的人共用一艘船，那么就这样安排。否则，最重的人无法与任何人配对，那么他们将自己独自乘一艘船。
     *
     * @param people
     * @param limit
     * @return 所需最小船数量
     */
    public int solution(int[] people, int limit) {
        // 首先进行排序
        Arrays.sort(people);
        // 左右指针
        int i = 0, j = people.length - 1;
        // 所需船数量
        int ans = 0;

        while (i <= j) {
            ans++;
            if (people[i] + people[j] <= limit) {
                // 如果两个人可以共用一条船，左指针向右移
                i++;
            }
            // 右指针向左移，可能存在可以共用救生艇的人，也可能比较重的人独自乘一艘船
            j--;
        }

        return ans;
    }
```

### 3、滑动窗口

这个名称可能大家就比较熟悉了，就是说有两个指针，一前一后组成滑动窗口，窗口的宽度可以是固定的，算法题相关的话大部分都是不固定宽度的，然后计算滑动窗口中元素的值。

我们可以通过下面这道题更为直观的了解。

```
#209. 长度最小的子数组
给定一个含有 n 个正整数的数组和一个正整数 target 。
找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

示例 1：
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。

示例 2：
输入：target = 4, nums = [1,4,4]
输出：1

示例 3：
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0

提示：
 1 <= target <= 109
 1 <= nums.length <= 105
 1 <= nums[i] <= 105
```

下面我们来看一下这道题目的做题思路，其实原理也很简单，我们创建两个指针，一个指针负责在前面探路，并不断累加遍历过的元素的值，当和大于等于我们的目标值时，后指针开始进行移动，判断去除当前值时，是否仍能满足我们的要求，直到不满足时后指针停止，前面指针继续移动，直到遍历结束。前指针和后指针之间的元素个数就是我们的滑动窗口的窗口大小，即最小子数组长度。

```java
		/**
     * 滑动窗口：就是通过不断调节子数组的起始位置和终止位置，进而得到我们想要的结果，滑动窗口也是双指针的一种。
     *
     * @param target 目标值
     * @param nums 数组
     * @return len 最小长度
     */
    public int minSubArrayLen(int target, int[] nums) {
        int len = nums.length;
        int windowLen = Integer.MAX_VALUE;
        // i 后指针
        int i = 0;
        int sum = 0;
        for (int j = 0; j < len; ++j) {
            // j 为前指针
            sum += nums[j];
            while (sum >= target) {
                // 比较求最小值
                windowLen = Math.min(windowLen, j - i + 1);
                // 移除当前值
                sum -= nums[i];
                // 移动后指针
                i++;
            }
        }
        return windowLen == Integer.MAX_VALUE ? 0 : windowLen;
    }
```

### 总结

对于查找某个重复值，判断是否有环，两个链表是否有相交节点等等这类的题目，我们都可以首先使用 **快慢指针** 来解决；对于有序的列表或者数组，我们可以优先考虑一下 **对撞指针** 来解决；**滑动窗口** 则很适用于求数组某个范围内元素的计算结果。

### 链接

- 参考：算法基地（袁小厨的算法基地）：https://github.com/chefyuan/algorithm-base
- LeetCode #141. 环形链表：https://leetcode-cn.com/problems/linked-list-cycle
- LeetCode #881. 救生艇：https://leetcode-cn.com/problems/boats-to-save-people
- LeetCode #26. 删除排序数组中的重复项：https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array
- LeetCode #209. 长度最小的子数组：https://leetcode-cn.com/problems/minimum-size-subarray-sum

