# Top 25 Java Problem Solving Interview Questions 2025

## Introduction

This guide covers the most frequently asked coding/problem-solving questions in Java interviews at top companies like Google, Amazon, Microsoft, Meta, Apple, and others. Each problem includes multiple solution approaches, time/space complexity analysis, and Java 8+ Stream solutions where applicable.

---

# String Problems

---

## 1. Reverse a String (Without Built-in Methods)

### Problem
Reverse a given string without using built-in reverse methods.

### Solutions

```java
public class ReverseString {
    
    // Method 1: Using char array and two pointers - O(n)
    public static String reverse(String str) {
        if (str == null || str.isEmpty()) return str;
        
        char[] chars = str.toCharArray();
        int left = 0, right = chars.length - 1;
        
        while (left < right) {
            char temp = chars[left];
            chars[left] = chars[right];
            chars[right] = temp;
            left++;
            right--;
        }
        return new String(chars);
    }
    
    // Method 2: Using StringBuilder - O(n)
    public static String reverseWithBuilder(String str) {
        if (str == null || str.isEmpty()) return str;
        
        StringBuilder sb = new StringBuilder();
        for (int i = str.length() - 1; i >= 0; i--) {
            sb.append(str.charAt(i));
        }
        return sb.toString();
    }
    
    // Method 3: Using recursion - O(n)
    public static String reverseRecursive(String str) {
        if (str == null || str.length() <= 1) return str;
        return reverseRecursive(str.substring(1)) + str.charAt(0);
    }
    
    // Method 4: Using Java 8 Stream
    public static String reverseStream(String str) {
        return new StringBuilder(str).reverse().toString();
    }
    
    public static void main(String[] args) {
        System.out.println(reverse("Hello World"));  // dlroW olleH
        System.out.println(reverseRecursive("Java")); // avaJ
    }
}
```

| Approach | Time | Space |
|----------|------|-------|
| Two Pointers | O(n) | O(n) |
| StringBuilder | O(n) | O(n) |
| Recursion | O(n) | O(n) stack |

---

## 2. Check if String is Palindrome

### Problem
Check if a given string reads the same forwards and backwards (ignoring case and non-alphanumeric characters).

### Solutions

```java
public class Palindrome {
    
    // Method 1: Two pointers - O(n) time, O(1) space
    public static boolean isPalindrome(String str) {
        if (str == null) return false;
        
        int left = 0, right = str.length() - 1;
        
        while (left < right) {
            // Skip non-alphanumeric from left
            while (left < right && !Character.isLetterOrDigit(str.charAt(left))) {
                left++;
            }
            // Skip non-alphanumeric from right
            while (left < right && !Character.isLetterOrDigit(str.charAt(right))) {
                right--;
            }
            
            if (Character.toLowerCase(str.charAt(left)) != 
                Character.toLowerCase(str.charAt(right))) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
    
    // Method 2: Using StringBuilder
    public static boolean isPalindromeBuilder(String str) {
        String cleaned = str.toLowerCase().replaceAll("[^a-z0-9]", "");
        String reversed = new StringBuilder(cleaned).reverse().toString();
        return cleaned.equals(reversed);
    }
    
    // Method 3: Using Java 8 Streams
    public static boolean isPalindromeStream(String str) {
        String cleaned = str.toLowerCase().replaceAll("[^a-z0-9]", "");
        return IntStream.range(0, cleaned.length() / 2)
            .allMatch(i -> cleaned.charAt(i) == cleaned.charAt(cleaned.length() - 1 - i));
    }
    
    public static void main(String[] args) {
        System.out.println(isPalindrome("A man, a plan, a canal: Panama")); // true
        System.out.println(isPalindrome("race a car")); // false
        System.out.println(isPalindrome("Was it a car or a cat I saw?")); // true
    }
}
```

---

## 3. Check if Two Strings are Anagrams

### Problem
Check if two strings contain the same characters with the same frequency.

### Solutions

```java
public class Anagram {
    
    // Method 1: Character count array - O(n) time, O(1) space
    public static boolean isAnagram(String s1, String s2) {
        if (s1 == null || s2 == null) return false;
        if (s1.length() != s2.length()) return false;
        
        int[] count = new int[26];
        
        for (int i = 0; i < s1.length(); i++) {
            count[Character.toLowerCase(s1.charAt(i)) - 'a']++;
            count[Character.toLowerCase(s2.charAt(i)) - 'a']--;
        }
        
        for (int c : count) {
            if (c != 0) return false;
        }
        return true;
    }
    
    // Method 2: Sorting - O(n log n) time
    public static boolean isAnagramSort(String s1, String s2) {
        if (s1.length() != s2.length()) return false;
        
        char[] arr1 = s1.toLowerCase().toCharArray();
        char[] arr2 = s2.toLowerCase().toCharArray();
        
        Arrays.sort(arr1);
        Arrays.sort(arr2);
        
        return Arrays.equals(arr1, arr2);
    }
    
    // Method 3: Using HashMap (supports Unicode)
    public static boolean isAnagramMap(String s1, String s2) {
        if (s1.length() != s2.length()) return false;
        
        Map<Character, Integer> map = new HashMap<>();
        
        for (char c : s1.toLowerCase().toCharArray()) {
            map.merge(c, 1, Integer::sum);
        }
        
        for (char c : s2.toLowerCase().toCharArray()) {
            int count = map.getOrDefault(c, 0);
            if (count == 0) return false;
            map.put(c, count - 1);
        }
        return true;
    }
    
    // Method 4: Using Streams
    public static boolean isAnagramStream(String s1, String s2) {
        if (s1.length() != s2.length()) return false;
        
        Map<Integer, Long> map1 = s1.toLowerCase().chars()
            .boxed()
            .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
            
        Map<Integer, Long> map2 = s2.toLowerCase().chars()
            .boxed()
            .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
            
        return map1.equals(map2);
    }
    
    public static void main(String[] args) {
        System.out.println(isAnagram("listen", "silent")); // true
        System.out.println(isAnagram("triangle", "integral")); // true
        System.out.println(isAnagram("hello", "world")); // false
    }
}
```

---

## 4. Find First Non-Repeated Character

### Problem
Find the first character in a string that appears only once.

### Solutions

```java
public class FirstNonRepeated {
    
    // Method 1: Using LinkedHashMap - O(n) time, O(k) space
    public static Character findFirst(String str) {
        if (str == null || str.isEmpty()) return null;
        
        Map<Character, Integer> map = new LinkedHashMap<>();
        
        for (char c : str.toCharArray()) {
            map.put(c, map.getOrDefault(c, 0) + 1);
        }
        
        for (Map.Entry<Character, Integer> entry : map.entrySet()) {
            if (entry.getValue() == 1) {
                return entry.getKey();
            }
        }
        return null;
    }
    
    // Method 2: Using indexOf and lastIndexOf - O(n²)
    public static Character findFirstSimple(String str) {
        for (int i = 0; i < str.length(); i++) {
            char c = str.charAt(i);
            if (str.indexOf(c) == str.lastIndexOf(c)) {
                return c;
            }
        }
        return null;
    }
    
    // Method 3: Using Streams
    public static Character findFirstStream(String str) {
        return str.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(c -> c, LinkedHashMap::new, Collectors.counting()))
            .entrySet()
            .stream()
            .filter(e -> e.getValue() == 1)
            .map(Map.Entry::getKey)
            .findFirst()
            .orElse(null);
    }
    
    public static void main(String[] args) {
        System.out.println(findFirst("swiss"));      // w
        System.out.println(findFirst("success"));    // u
        System.out.println(findFirst("aabbcc"));     // null
    }
}
```

---

## 5. Find Duplicate Characters in String

### Problem
Find all characters that appear more than once in a string.

### Solutions

```java
public class DuplicateCharacters {
    
    // Method 1: Using HashSet - O(n)
    public static Set<Character> findDuplicates(String str) {
        Set<Character> seen = new HashSet<>();
        Set<Character> duplicates = new LinkedHashSet<>();
        
        for (char c : str.toLowerCase().toCharArray()) {
            if (c == ' ') continue;
            if (!seen.add(c)) {
                duplicates.add(c);
            }
        }
        return duplicates;
    }
    
    // Method 2: With count using HashMap
    public static Map<Character, Integer> findDuplicatesWithCount(String str) {
        Map<Character, Integer> charCount = new LinkedHashMap<>();
        
        for (char c : str.toLowerCase().toCharArray()) {
            if (c == ' ') continue;
            charCount.merge(c, 1, Integer::sum);
        }
        
        charCount.entrySet().removeIf(e -> e.getValue() < 2);
        return charCount;
    }
    
    // Method 3: Using Streams
    public static Map<Character, Long> findDuplicatesStream(String str) {
        return str.toLowerCase().chars()
            .filter(c -> c != ' ')
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(c -> c, LinkedHashMap::new, Collectors.counting()))
            .entrySet()
            .stream()
            .filter(e -> e.getValue() > 1)
            .collect(Collectors.toMap(
                Map.Entry::getKey, 
                Map.Entry::getValue,
                (a, b) -> a,
                LinkedHashMap::new
            ));
    }
    
    public static void main(String[] args) {
        System.out.println(findDuplicates("programming")); // [r, g, m]
        System.out.println(findDuplicatesWithCount("java interview")); // {a=2, i=2}
    }
}
```

---

## 6. Longest Substring Without Repeating Characters

### Problem
Find the length of the longest substring without repeating characters.

### Solutions

```java
public class LongestSubstringWithoutRepeat {
    
    // Method 1: Sliding Window with HashSet - O(n)
    public static int lengthOfLongestSubstring(String s) {
        if (s == null || s.isEmpty()) return 0;
        
        Set<Character> set = new HashSet<>();
        int maxLen = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            
            while (set.contains(c)) {
                set.remove(s.charAt(left));
                left++;
            }
            
            set.add(c);
            maxLen = Math.max(maxLen, right - left + 1);
        }
        
        return maxLen;
    }
    
    // Method 2: Using HashMap for optimization - O(n)
    public static int lengthOfLongestSubstringOptimized(String s) {
        if (s == null || s.isEmpty()) return 0;
        
        Map<Character, Integer> map = new HashMap<>();
        int maxLen = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            
            if (map.containsKey(c)) {
                left = Math.max(left, map.get(c) + 1);
            }
            
            map.put(c, right);
            maxLen = Math.max(maxLen, right - left + 1);
        }
        
        return maxLen;
    }
    
    // Get the actual substring
    public static String getLongestSubstring(String s) {
        if (s == null || s.isEmpty()) return "";
        
        Map<Character, Integer> map = new HashMap<>();
        int maxLen = 0, maxStart = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            
            if (map.containsKey(c)) {
                left = Math.max(left, map.get(c) + 1);
            }
            
            map.put(c, right);
            
            if (right - left + 1 > maxLen) {
                maxLen = right - left + 1;
                maxStart = left;
            }
        }
        
        return s.substring(maxStart, maxStart + maxLen);
    }
    
    public static void main(String[] args) {
        System.out.println(lengthOfLongestSubstring("abcabcbb")); // 3 ("abc")
        System.out.println(lengthOfLongestSubstring("bbbbb"));    // 1 ("b")
        System.out.println(lengthOfLongestSubstring("pwwkew"));   // 3 ("wke")
        System.out.println(getLongestSubstring("abcabcbb"));      // "abc"
    }
}
```

---

# Array Problems

---

## 7. Find Second Largest Element

### Problem
Find the second largest element in an array.

### Solutions

```java
public class SecondLargest {
    
    // Method 1: Single pass - O(n) time, O(1) space
    public static int findSecondLargest(int[] arr) {
        if (arr == null || arr.length < 2) {
            throw new IllegalArgumentException("Array must have at least 2 elements");
        }
        
        int first = Integer.MIN_VALUE;
        int second = Integer.MIN_VALUE;
        
        for (int num : arr) {
            if (num > first) {
                second = first;
                first = num;
            } else if (num > second && num != first) {
                second = num;
            }
        }
        
        if (second == Integer.MIN_VALUE) {
            throw new IllegalArgumentException("No second largest element exists");
        }
        
        return second;
    }
    
    // Method 2: Using Streams
    public static int findSecondLargestStream(int[] arr) {
        return Arrays.stream(arr)
            .boxed()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("No second largest"));
    }
    
    // Method 3: Using PriorityQueue
    public static int findSecondLargestHeap(int[] arr) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        Set<Integer> seen = new HashSet<>();
        
        for (int num : arr) {
            if (seen.add(num)) {
                minHeap.offer(num);
                if (minHeap.size() > 2) {
                    minHeap.poll();
                }
            }
        }
        
        return minHeap.peek();
    }
    
    public static void main(String[] args) {
        int[] arr = {12, 35, 1, 10, 34, 1};
        System.out.println(findSecondLargest(arr)); // 34
        System.out.println(findSecondLargestStream(arr)); // 34
    }
}
```

---

## 8. Find Missing Number in Array (1 to n)

### Problem
Given an array containing n distinct numbers from 0 to n, find the missing number.

### Solutions

```java
public class MissingNumber {
    
    // Method 1: Sum formula - O(n) time, O(1) space
    public static int findMissing(int[] arr, int n) {
        int expectedSum = n * (n + 1) / 2;
        int actualSum = 0;
        
        for (int num : arr) {
            actualSum += num;
        }
        
        return expectedSum - actualSum;
    }
    
    // Method 2: XOR approach (handles overflow) - O(n) time, O(1) space
    public static int findMissingXOR(int[] arr, int n) {
        int xor = 0;
        
        // XOR all numbers from 1 to n
        for (int i = 1; i <= n; i++) {
            xor ^= i;
        }
        
        // XOR all numbers in array
        for (int num : arr) {
            xor ^= num;
        }
        
        return xor;
    }
    
    // Method 3: Using Streams
    public static int findMissingStream(int[] arr, int n) {
        int expectedSum = IntStream.rangeClosed(1, n).sum();
        int actualSum = Arrays.stream(arr).sum();
        return expectedSum - actualSum;
    }
    
    // Method 4: Using Set (for multiple missing numbers)
    public static List<Integer> findAllMissing(int[] arr, int n) {
        Set<Integer> set = Arrays.stream(arr).boxed().collect(Collectors.toSet());
        
        return IntStream.rangeClosed(1, n)
            .filter(i -> !set.contains(i))
            .boxed()
            .collect(Collectors.toList());
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 4, 5, 6};
        System.out.println(findMissing(arr, 6));    // 3
        System.out.println(findMissingXOR(arr, 6)); // 3
        
        int[] arr2 = {1, 3, 6, 8, 10};
        System.out.println(findAllMissing(arr2, 10)); // [2, 4, 5, 7, 9]
    }
}
```

---

## 9. Two Sum - Find Pair with Given Sum

### Problem
Find two numbers in an array that add up to a target sum.

### Solutions

```java
public class TwoSum {
    
    // Method 1: HashMap - O(n) time, O(n) space
    public static int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            if (map.containsKey(complement)) {
                return new int[] {map.get(complement), i};
            }
            map.put(nums[i], i);
        }
        
        return new int[] {}; // No solution
    }
    
    // Method 2: Two Pointers (for sorted array) - O(n) time, O(1) space
    public static int[] twoSumSorted(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        
        while (left < right) {
            int sum = nums[left] + nums[right];
            
            if (sum == target) {
                return new int[] {left, right};
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
        
        return new int[] {};
    }
    
    // Return actual values instead of indices
    public static int[] twoSumValues(int[] nums, int target) {
        Set<Integer> seen = new HashSet<>();
        
        for (int num : nums) {
            int complement = target - num;
            if (seen.contains(complement)) {
                return new int[] {complement, num};
            }
            seen.add(num);
        }
        
        return new int[] {};
    }
    
    // Find all pairs with given sum
    public static List<int[]> findAllPairs(int[] nums, int target) {
        List<int[]> result = new ArrayList<>();
        Map<Integer, Integer> map = new HashMap<>();
        
        for (int num : nums) {
            int complement = target - num;
            int count = map.getOrDefault(complement, 0);
            
            for (int i = 0; i < count; i++) {
                result.add(new int[] {complement, num});
            }
            
            map.merge(num, 1, Integer::sum);
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        int[] nums = {2, 7, 11, 15};
        System.out.println(Arrays.toString(twoSum(nums, 9))); // [0, 1]
        
        int[] sorted = {1, 2, 3, 4, 5, 6};
        System.out.println(Arrays.toString(twoSumSorted(sorted, 9))); // [2, 5]
    }
}
```

---

## 10. Move Zeroes to End

### Problem
Move all zeros in an array to the end while maintaining the relative order of non-zero elements.

### Solutions

```java
public class MoveZeroes {
    
    // Method 1: Two pointer swap - O(n) time, O(1) space
    public static void moveZeroes(int[] nums) {
        int insertPos = 0;
        
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != 0) {
                // Swap
                int temp = nums[insertPos];
                nums[insertPos] = nums[i];
                nums[i] = temp;
                insertPos++;
            }
        }
    }
    
    // Method 2: Copy non-zeros, then fill zeros
    public static void moveZeroesSimple(int[] nums) {
        int insertPos = 0;
        
        // Move all non-zero elements to front
        for (int num : nums) {
            if (num != 0) {
                nums[insertPos++] = num;
            }
        }
        
        // Fill remaining with zeros
        while (insertPos < nums.length) {
            nums[insertPos++] = 0;
        }
    }
    
    // Using Streams (creates new array)
    public static int[] moveZeroesStream(int[] nums) {
        int[] nonZeros = Arrays.stream(nums)
            .filter(n -> n != 0)
            .toArray();
        
        int[] result = new int[nums.length];
        System.arraycopy(nonZeros, 0, result, 0, nonZeros.length);
        
        return result;
    }
    
    public static void main(String[] args) {
        int[] arr = {0, 1, 0, 3, 12};
        moveZeroes(arr);
        System.out.println(Arrays.toString(arr)); // [1, 3, 12, 0, 0]
    }
}
```

---

## 11. Find Duplicate in Array

### Problem
Find the duplicate number in an array containing n+1 integers where each integer is between 1 and n.

### Solutions

```java
public class FindDuplicate {
    
    // Method 1: HashSet - O(n) time, O(n) space
    public static int findDuplicateSet(int[] nums) {
        Set<Integer> seen = new HashSet<>();
        
        for (int num : nums) {
            if (!seen.add(num)) {
                return num;
            }
        }
        return -1;
    }
    
    // Method 2: Floyd's Cycle Detection - O(n) time, O(1) space
    public static int findDuplicate(int[] nums) {
        int slow = nums[0];
        int fast = nums[0];
        
        // Phase 1: Find intersection point
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);
        
        // Phase 2: Find entrance to cycle
        slow = nums[0];
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        
        return fast;
    }
    
    // Method 3: Marking visited (modifies array) - O(n) time, O(1) space
    public static int findDuplicateMarking(int[] nums) {
        for (int num : nums) {
            int index = Math.abs(num);
            if (nums[index] < 0) {
                return index;
            }
            nums[index] = -nums[index];
        }
        return -1;
    }
    
    // Find all duplicates
    public static List<Integer> findAllDuplicates(int[] nums) {
        List<Integer> result = new ArrayList<>();
        
        for (int num : nums) {
            int index = Math.abs(num) - 1;
            if (nums[index] < 0) {
                result.add(Math.abs(num));
            } else {
                nums[index] = -nums[index];
            }
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        int[] nums = {1, 3, 4, 2, 2};
        System.out.println(findDuplicate(nums)); // 2
        
        int[] nums2 = {4, 3, 2, 7, 8, 2, 3, 1};
        System.out.println(findAllDuplicates(nums2)); // [2, 3]
    }
}
```

---

## 12. Rotate Array by K Positions

### Problem
Rotate an array to the right by k steps.

### Solutions

```java
public class RotateArray {
    
    // Method 1: Reverse approach - O(n) time, O(1) space
    public static void rotate(int[] nums, int k) {
        k = k % nums.length;
        if (k == 0) return;
        
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.length - 1);
    }
    
    private static void reverse(int[] nums, int start, int end) {
        while (start < end) {
            int temp = nums[start];
            nums[start] = nums[end];
            nums[end] = temp;
            start++;
            end--;
        }
    }
    
    // Method 2: Using extra array - O(n) time, O(n) space
    public static void rotateExtraSpace(int[] nums, int k) {
        k = k % nums.length;
        int[] temp = new int[nums.length];
        
        for (int i = 0; i < nums.length; i++) {
            temp[(i + k) % nums.length] = nums[i];
        }
        
        System.arraycopy(temp, 0, nums, 0, nums.length);
    }
    
    // Method 3: Cyclic replacements - O(n) time, O(1) space
    public static void rotateCyclic(int[] nums, int k) {
        k = k % nums.length;
        int count = 0;
        
        for (int start = 0; count < nums.length; start++) {
            int current = start;
            int prev = nums[start];
            
            do {
                int next = (current + k) % nums.length;
                int temp = nums[next];
                nums[next] = prev;
                prev = temp;
                current = next;
                count++;
            } while (start != current);
        }
    }
    
    // Left rotation
    public static void rotateLeft(int[] nums, int k) {
        k = k % nums.length;
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.length - 1);
        reverse(nums, 0, nums.length - 1);
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5, 6, 7};
        rotate(arr, 3);
        System.out.println(Arrays.toString(arr)); // [5, 6, 7, 1, 2, 3, 4]
    }
}
```

---

## 13. Maximum Subarray Sum (Kadane's Algorithm)

### Problem
Find the contiguous subarray with the largest sum.

### Solutions

```java
public class MaxSubarraySum {
    
    // Kadane's Algorithm - O(n) time, O(1) space
    public static int maxSubArray(int[] nums) {
        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];
        
        for (int i = 1; i < nums.length; i++) {
            maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
            maxSoFar = Math.max(maxSoFar, maxEndingHere);
        }
        
        return maxSoFar;
    }
    
    // Return subarray indices as well
    public static int[] maxSubArrayWithIndices(int[] nums) {
        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];
        int start = 0, end = 0, tempStart = 0;
        
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] > maxEndingHere + nums[i]) {
                maxEndingHere = nums[i];
                tempStart = i;
            } else {
                maxEndingHere = maxEndingHere + nums[i];
            }
            
            if (maxEndingHere > maxSoFar) {
                maxSoFar = maxEndingHere;
                start = tempStart;
                end = i;
            }
        }
        
        return new int[] {maxSoFar, start, end};
    }
    
    // Using Divide and Conquer - O(n log n)
    public static int maxSubArrayDivideConquer(int[] nums) {
        return maxSubArrayHelper(nums, 0, nums.length - 1);
    }
    
    private static int maxSubArrayHelper(int[] nums, int left, int right) {
        if (left == right) return nums[left];
        
        int mid = left + (right - left) / 2;
        
        int leftMax = maxSubArrayHelper(nums, left, mid);
        int rightMax = maxSubArrayHelper(nums, mid + 1, right);
        int crossMax = maxCrossingSum(nums, left, mid, right);
        
        return Math.max(Math.max(leftMax, rightMax), crossMax);
    }
    
    private static int maxCrossingSum(int[] nums, int left, int mid, int right) {
        int leftSum = Integer.MIN_VALUE;
        int sum = 0;
        for (int i = mid; i >= left; i--) {
            sum += nums[i];
            leftSum = Math.max(leftSum, sum);
        }
        
        int rightSum = Integer.MIN_VALUE;
        sum = 0;
        for (int i = mid + 1; i <= right; i++) {
            sum += nums[i];
            rightSum = Math.max(rightSum, sum);
        }
        
        return leftSum + rightSum;
    }
    
    public static void main(String[] args) {
        int[] nums = {-2, 1, -3, 4, -1, 2, 1, -5, 4};
        System.out.println(maxSubArray(nums)); // 6 (subarray [4, -1, 2, 1])
        
        int[] result = maxSubArrayWithIndices(nums);
        System.out.println("Max: " + result[0] + ", Start: " + result[1] + ", End: " + result[2]);
    }
}
```

---

# Number Problems

---

## 14. Check if Number is Prime

### Problem
Determine if a given number is prime.

### Solutions

```java
public class PrimeNumber {
    
    // Method 1: Optimized trial division - O(√n)
    public static boolean isPrime(int n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0 || n % 3 == 0) return false;
        
        // Check divisibility by 6k ± 1
        for (int i = 5; i * i <= n; i += 6) {
            if (n % i == 0 || n % (i + 2) == 0) {
                return false;
            }
        }
        return true;
    }
    
    // Method 2: Using Streams
    public static boolean isPrimeStream(int n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0) return false;
        
        return IntStream.iterate(3, i -> i + 2)
            .takeWhile(i -> i * i <= n)
            .noneMatch(i -> n % i == 0);
    }
    
    // Sieve of Eratosthenes - O(n log log n)
    public static List<Integer> sieveOfEratosthenes(int n) {
        boolean[] isPrime = new boolean[n + 1];
        Arrays.fill(isPrime, true);
        isPrime[0] = isPrime[1] = false;
        
        for (int i = 2; i * i <= n; i++) {
            if (isPrime[i]) {
                for (int j = i * i; j <= n; j += i) {
                    isPrime[j] = false;
                }
            }
        }
        
        List<Integer> primes = new ArrayList<>();
        for (int i = 2; i <= n; i++) {
            if (isPrime[i]) primes.add(i);
        }
        return primes;
    }
    
    // Count primes less than n
    public static int countPrimes(int n) {
        if (n <= 2) return 0;
        
        boolean[] notPrime = new boolean[n];
        int count = 0;
        
        for (int i = 2; i < n; i++) {
            if (!notPrime[i]) {
                count++;
                for (long j = (long) i * i; j < n; j += i) {
                    notPrime[(int) j] = true;
                }
            }
        }
        
        return count;
    }
    
    public static void main(String[] args) {
        System.out.println(isPrime(17));    // true
        System.out.println(isPrime(18));    // false
        System.out.println(isPrime(997));   // true
        System.out.println(sieveOfEratosthenes(30)); // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
        System.out.println(countPrimes(100)); // 25
    }
}
```

---

## 15. Fibonacci Series

### Problem
Generate Fibonacci numbers.

### Solutions

```java
public class Fibonacci {
    
    // Method 1: Iterative - O(n) time, O(1) space
    public static long fibIterative(int n) {
        if (n <= 1) return n;
        
        long prev = 0, curr = 1;
        
        for (int i = 2; i <= n; i++) {
            long next = prev + curr;
            prev = curr;
            curr = next;
        }
        return curr;
    }
    
    // Method 2: Recursive with Memoization - O(n) time, O(n) space
    private static Map<Integer, Long> memo = new HashMap<>();
    
    public static long fibMemoized(int n) {
        if (n <= 1) return n;
        
        return memo.computeIfAbsent(n, key -> 
            fibMemoized(key - 1) + fibMemoized(key - 2));
    }
    
    // Method 3: Dynamic Programming - O(n) time, O(n) space
    public static long fibDP(int n) {
        if (n <= 1) return n;
        
        long[] dp = new long[n + 1];
        dp[0] = 0;
        dp[1] = 1;
        
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // Method 4: Using Streams - Generate series
    public static List<Long> fibSeries(int n) {
        return Stream.iterate(new long[]{0, 1}, arr -> new long[]{arr[1], arr[0] + arr[1]})
            .limit(n)
            .map(arr -> arr[0])
            .collect(Collectors.toList());
    }
    
    // Method 5: Matrix Exponentiation - O(log n)
    public static long fibMatrix(int n) {
        if (n <= 1) return n;
        
        long[][] matrix = {{1, 1}, {1, 0}};
        long[][] result = matrixPower(matrix, n - 1);
        
        return result[0][0];
    }
    
    private static long[][] matrixPower(long[][] matrix, int n) {
        long[][] result = {{1, 0}, {0, 1}}; // Identity matrix
        
        while (n > 0) {
            if ((n & 1) == 1) {
                result = multiplyMatrix(result, matrix);
            }
            matrix = multiplyMatrix(matrix, matrix);
            n >>= 1;
        }
        
        return result;
    }
    
    private static long[][] multiplyMatrix(long[][] a, long[][] b) {
        return new long[][] {
            {a[0][0] * b[0][0] + a[0][1] * b[1][0], a[0][0] * b[0][1] + a[0][1] * b[1][1]},
            {a[1][0] * b[0][0] + a[1][1] * b[1][0], a[1][0] * b[0][1] + a[1][1] * b[1][1]}
        };
    }
    
    public static void main(String[] args) {
        System.out.println(fibIterative(10));   // 55
        System.out.println(fibMemoized(50));    // 12586269025
        System.out.println(fibSeries(10));      // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
        System.out.println(fibMatrix(10));      // 55
    }
}
```

---

## 16. Reverse a Number

### Problem
Reverse the digits of an integer.

### Solutions

```java
public class ReverseNumber {
    
    // Method 1: Mathematical approach - O(log n)
    public static int reverse(int num) {
        int reversed = 0;
        boolean negative = num < 0;
        num = Math.abs(num);
        
        while (num > 0) {
            int digit = num % 10;
            
            // Check for overflow
            if (reversed > (Integer.MAX_VALUE - digit) / 10) {
                return 0;
            }
            
            reversed = reversed * 10 + digit;
            num /= 10;
        }
        
        return negative ? -reversed : reversed;
    }
    
    // Method 2: Using String
    public static int reverseString(int num) {
        boolean negative = num < 0;
        String str = String.valueOf(Math.abs(num));
        String reversed = new StringBuilder(str).reverse().toString();
        
        try {
            int result = Integer.parseInt(reversed);
            return negative ? -result : result;
        } catch (NumberFormatException e) {
            return 0; // Overflow
        }
    }
    
    // Method 3: Using recursion
    public static int reverseRecursive(int num, int reversed) {
        if (num == 0) return reversed;
        return reverseRecursive(num / 10, reversed * 10 + num % 10);
    }
    
    public static void main(String[] args) {
        System.out.println(reverse(12345));     // 54321
        System.out.println(reverse(-12345));    // -54321
        System.out.println(reverse(120));       // 21
        System.out.println(reverse(1534236469)); // 0 (overflow)
    }
}
```

---

## 17. Check Palindrome Number

### Problem
Check if a number reads the same forwards and backwards.

### Solutions

```java
public class PalindromeNumber {
    
    // Method 1: Reverse half the number - O(log n)
    public static boolean isPalindrome(int x) {
        // Negative numbers and numbers ending in 0 (except 0 itself) are not palindromes
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        
        int reversed = 0;
        while (x > reversed) {
            reversed = reversed * 10 + x % 10;
            x /= 10;
        }
        
        // For odd length numbers, we need to handle middle digit
        return x == reversed || x == reversed / 10;
    }
    
    // Method 2: Full reverse
    public static boolean isPalindromeFullReverse(int x) {
        if (x < 0) return false;
        
        int original = x;
        int reversed = 0;
        
        while (x > 0) {
            reversed = reversed * 10 + x % 10;
            x /= 10;
        }
        
        return original == reversed;
    }
    
    // Method 3: Using String
    public static boolean isPalindromeString(int x) {
        String str = String.valueOf(x);
        return str.equals(new StringBuilder(str).reverse().toString());
    }
    
    // Method 4: Two pointer on digits
    public static boolean isPalindromeTwoPointer(int x) {
        if (x < 0) return false;
        
        String str = String.valueOf(x);
        int left = 0, right = str.length() - 1;
        
        while (left < right) {
            if (str.charAt(left++) != str.charAt(right--)) {
                return false;
            }
        }
        return true;
    }
    
    public static void main(String[] args) {
        System.out.println(isPalindrome(121));    // true
        System.out.println(isPalindrome(-121));   // false
        System.out.println(isPalindrome(10));     // false
        System.out.println(isPalindrome(12321));  // true
    }
}
```

---

## 18. GCD and LCM

### Problem
Find the Greatest Common Divisor and Least Common Multiple of two numbers.

### Solutions

```java
public class GcdLcm {
    
    // GCD using Euclidean Algorithm - O(log(min(a,b)))
    public static int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return Math.abs(a);
    }
    
    // GCD using Recursion
    public static int gcdRecursive(int a, int b) {
        if (b == 0) return Math.abs(a);
        return gcdRecursive(b, a % b);
    }
    
    // GCD using Streams (for multiple numbers)
    public static int gcdMultiple(int... numbers) {
        return Arrays.stream(numbers)
            .reduce(0, GcdLcm::gcd);
    }
    
    // LCM using GCD
    public static long lcm(int a, int b) {
        return Math.abs((long) a * b) / gcd(a, b);
    }
    
    // LCM for multiple numbers
    public static long lcmMultiple(int... numbers) {
        return Arrays.stream(numbers)
            .mapToLong(i -> i)
            .reduce(1, (a, b) -> a * b / gcd((int) a, (int) b));
    }
    
    // Extended Euclidean Algorithm (finds x, y such that ax + by = gcd(a,b))
    public static int[] extendedGcd(int a, int b) {
        if (b == 0) {
            return new int[] {a, 1, 0};
        }
        
        int[] result = extendedGcd(b, a % b);
        int gcd = result[0];
        int x = result[2];
        int y = result[1] - (a / b) * result[2];
        
        return new int[] {gcd, x, y};
    }
    
    public static void main(String[] args) {
        System.out.println(gcd(48, 18));        // 6
        System.out.println(lcm(4, 6));          // 12
        System.out.println(gcdMultiple(12, 18, 24)); // 6
        System.out.println(lcmMultiple(4, 6, 8));    // 24
    }
}
```

---

# Data Structure Problems

---

## 19. Reverse a Linked List

### Problem
Reverse a singly linked list.

### Solutions

```java
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
    }
}

public class ReverseLinkedList {
    
    // Method 1: Iterative - O(n) time, O(1) space
    public static ListNode reverseIterative(ListNode head) {
        ListNode prev = null;
        ListNode current = head;
        
        while (current != null) {
            ListNode next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        
        return prev;
    }
    
    // Method 2: Recursive - O(n) time, O(n) space
    public static ListNode reverseRecursive(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        
        ListNode newHead = reverseRecursive(head.next);
        head.next.next = head;
        head.next = null;
        
        return newHead;
    }
    
    // Method 3: Reverse between positions m and n
    public static ListNode reverseBetween(ListNode head, int m, int n) {
        if (head == null || m == n) return head;
        
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prev = dummy;
        
        // Move to position m
        for (int i = 1; i < m; i++) {
            prev = prev.next;
        }
        
        ListNode current = prev.next;
        
        // Reverse n-m nodes
        for (int i = 0; i < n - m; i++) {
            ListNode next = current.next;
            current.next = next.next;
            next.next = prev.next;
            prev.next = next;
        }
        
        return dummy.next;
    }
    
    // Helper: Create linked list from array
    public static ListNode createList(int[] arr) {
        if (arr.length == 0) return null;
        
        ListNode head = new ListNode(arr[0]);
        ListNode current = head;
        
        for (int i = 1; i < arr.length; i++) {
            current.next = new ListNode(arr[i]);
            current = current.next;
        }
        
        return head;
    }
    
    // Helper: Print linked list
    public static void printList(ListNode head) {
        StringBuilder sb = new StringBuilder();
        while (head != null) {
            sb.append(head.val).append(" -> ");
            head = head.next;
        }
        sb.append("null");
        System.out.println(sb);
    }
    
    public static void main(String[] args) {
        ListNode head = createList(new int[]{1, 2, 3, 4, 5});
        printList(head);  // 1 -> 2 -> 3 -> 4 -> 5 -> null
        
        head = reverseIterative(head);
        printList(head);  // 5 -> 4 -> 3 -> 2 -> 1 -> null
    }
}
```

---

## 20. Detect Cycle in Linked List

### Problem
Determine if a linked list has a cycle.

### Solutions

```java
public class DetectCycle {
    
    // Method 1: Floyd's Cycle Detection - O(n) time, O(1) space
    public static boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                return true;
            }
        }
        
        return false;
    }
    
    // Find the starting node of cycle
    public static ListNode detectCycleStart(ListNode head) {
        if (head == null || head.next == null) {
            return null;
        }
        
        ListNode slow = head;
        ListNode fast = head;
        
        // Detect cycle
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                // Cycle found, find start
                slow = head;
                while (slow != fast) {
                    slow = slow.next;
                    fast = fast.next;
                }
                return slow;
            }
        }
        
        return null;
    }
    
    // Find cycle length
    public static int cycleLength(ListNode head) {
        ListNode meetingPoint = detectCycleStart(head);
        
        if (meetingPoint == null) return 0;
        
        int length = 1;
        ListNode current = meetingPoint.next;
        
        while (current != meetingPoint) {
            length++;
            current = current.next;
        }
        
        return length;
    }
    
    // Method 2: Using HashSet - O(n) time, O(n) space
    public static boolean hasCycleSet(ListNode head) {
        Set<ListNode> visited = new HashSet<>();
        
        while (head != null) {
            if (!visited.add(head)) {
                return true;
            }
            head = head.next;
        }
        
        return false;
    }
}
```

---

## 21. Valid Parentheses

### Problem
Check if a string of parentheses is valid.

### Solutions

```java
public class ValidParentheses {
    
    // Method 1: Using Stack - O(n) time, O(n) space
    public static boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        
        Map<Character, Character> mapping = Map.of(
            ')', '(',
            '}', '{',
            ']', '['
        );
        
        for (char c : s.toCharArray()) {
            if (mapping.containsKey(c)) {
                if (stack.isEmpty() || stack.pop() != mapping.get(c)) {
                    return false;
                }
            } else {
                stack.push(c);
            }
        }
        
        return stack.isEmpty();
    }
    
    // Method 2: Without Map
    public static boolean isValidSimple(String s) {
        Stack<Character> stack = new Stack<>();
        
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) return false;
                
                char top = stack.pop();
                if ((c == ')' && top != '(') ||
                    (c == '}' && top != '{') ||
                    (c == ']' && top != '[')) {
                    return false;
                }
            }
        }
        
        return stack.isEmpty();
    }
    
    // Method 3: Using Deque (preferred over Stack)
    public static boolean isValidDeque(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        
        for (char c : s.toCharArray()) {
            if (c == '(') stack.push(')');
            else if (c == '{') stack.push('}');
            else if (c == '[') stack.push(']');
            else if (stack.isEmpty() || stack.pop() != c) {
                return false;
            }
        }
        
        return stack.isEmpty();
    }
    
    // Generate all valid parentheses combinations
    public static List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(result, new StringBuilder(), 0, 0, n);
        return result;
    }
    
    private static void backtrack(List<String> result, StringBuilder current, 
                                  int open, int close, int max) {
        if (current.length() == max * 2) {
            result.add(current.toString());
            return;
        }
        
        if (open < max) {
            current.append('(');
            backtrack(result, current, open + 1, close, max);
            current.deleteCharAt(current.length() - 1);
        }
        
        if (close < open) {
            current.append(')');
            backtrack(result, current, open, close + 1, max);
            current.deleteCharAt(current.length() - 1);
        }
    }
    
    public static void main(String[] args) {
        System.out.println(isValid("()"));        // true
        System.out.println(isValid("()[]{}"));    // true
        System.out.println(isValid("(]"));        // false
        System.out.println(isValid("([)]"));      // false
        System.out.println(isValid("{[]}"));      // true
        
        System.out.println(generateParenthesis(3));
        // ["((()))", "(()())", "(())()", "()(())", "()()()"]
    }
}
```

---

## 22. Merge Two Sorted Lists

### Problem
Merge two sorted linked lists into one sorted list.

### Solutions

```java
public class MergeSortedLists {
    
    // Method 1: Iterative - O(n+m) time, O(1) space
    public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode current = dummy;
        
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                current.next = l1;
                l1 = l1.next;
            } else {
                current.next = l2;
                l2 = l2.next;
            }
            current = current.next;
        }
        
        // Attach remaining nodes
        current.next = (l1 != null) ? l1 : l2;
        
        return dummy.next;
    }
    
    // Method 2: Recursive - O(n+m) time, O(n+m) space
    public static ListNode mergeTwoListsRecursive(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;
        
        if (l1.val <= l2.val) {
            l1.next = mergeTwoListsRecursive(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoListsRecursive(l1, l2.next);
            return l2;
        }
    }
    
    // Merge K sorted lists using PriorityQueue - O(N log k)
    public static ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        
        PriorityQueue<ListNode> pq = new PriorityQueue<>(
            Comparator.comparingInt(a -> a.val)
        );
        
        for (ListNode list : lists) {
            if (list != null) {
                pq.offer(list);
            }
        }
        
        ListNode dummy = new ListNode(0);
        ListNode current = dummy;
        
        while (!pq.isEmpty()) {
            ListNode node = pq.poll();
            current.next = node;
            current = current.next;
            
            if (node.next != null) {
                pq.offer(node.next);
            }
        }
        
        return dummy.next;
    }
}
```

---

## 23. Find Kth Largest Element

### Problem
Find the kth largest element in an unsorted array.

### Solutions

```java
public class KthLargest {
    
    // Method 1: Using Min Heap - O(n log k) time, O(k) space
    public static int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        
        for (int num : nums) {
            minHeap.offer(num);
            if (minHeap.size() > k) {
                minHeap.poll();
            }
        }
        
        return minHeap.peek();
    }
    
    // Method 2: QuickSelect - O(n) average, O(n²) worst
    public static int quickSelect(int[] nums, int k) {
        k = nums.length - k; // Convert to kth smallest
        int lo = 0, hi = nums.length - 1;
        
        while (lo < hi) {
            int pivotIndex = partition(nums, lo, hi);
            
            if (pivotIndex < k) {
                lo = pivotIndex + 1;
            } else if (pivotIndex > k) {
                hi = pivotIndex - 1;
            } else {
                break;
            }
        }
        
        return nums[k];
    }
    
    private static int partition(int[] nums, int lo, int hi) {
        int pivot = nums[hi];
        int i = lo;
        
        for (int j = lo; j < hi; j++) {
            if (nums[j] <= pivot) {
                swap(nums, i, j);
                i++;
            }
        }
        
        swap(nums, i, hi);
        return i;
    }
    
    private static void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    // Method 3: Using sorting - O(n log n)
    public static int findKthLargestSort(int[] nums, int k) {
        Arrays.sort(nums);
        return nums[nums.length - k];
    }
    
    // Method 4: Using Streams
    public static int findKthLargestStream(int[] nums, int k) {
        return Arrays.stream(nums)
            .boxed()
            .sorted(Comparator.reverseOrder())
            .skip(k - 1)
            .findFirst()
            .orElseThrow();
    }
    
    public static void main(String[] args) {
        int[] nums = {3, 2, 1, 5, 6, 4};
        System.out.println(findKthLargest(nums, 2));  // 5
        
        int[] nums2 = {3, 2, 3, 1, 2, 4, 5, 5, 6};
        System.out.println(findKthLargest(nums2, 4)); // 4
    }
}
```

---

## 24. Binary Search

### Problem
Search for a target value in a sorted array.

### Solutions

```java
public class BinarySearch {
    
    // Method 1: Iterative - O(log n) time, O(1) space
    public static int binarySearch(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2; // Avoid overflow
            
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return -1;
    }
    
    // Method 2: Recursive - O(log n) time, O(log n) space
    public static int binarySearchRecursive(int[] nums, int target) {
        return binarySearchHelper(nums, target, 0, nums.length - 1);
    }
    
    private static int binarySearchHelper(int[] nums, int target, int left, int right) {
        if (left > right) return -1;
        
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) {
            return binarySearchHelper(nums, target, mid + 1, right);
        }
        return binarySearchHelper(nums, target, left, mid - 1);
    }
    
    // Find first occurrence
    public static int findFirst(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        int result = -1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                result = mid;
                right = mid - 1; // Continue searching left
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return result;
    }
    
    // Find last occurrence
    public static int findLast(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        int result = -1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                result = mid;
                left = mid + 1; // Continue searching right
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return result;
    }
    
    // Count occurrences
    public static int countOccurrences(int[] nums, int target) {
        int first = findFirst(nums, target);
        if (first == -1) return 0;
        
        int last = findLast(nums, target);
        return last - first + 1;
    }
    
    // Search in rotated sorted array
    public static int searchRotated(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) return mid;
            
            // Left half is sorted
            if (nums[left] <= nums[mid]) {
                if (target >= nums[left] && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            }
            // Right half is sorted
            else {
                if (target > nums[mid] && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        
        return -1;
    }
    
    public static void main(String[] args) {
        int[] nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        System.out.println(binarySearch(nums, 7));  // 6
        
        int[] repeated = {1, 2, 2, 2, 3, 4, 5};
        System.out.println(findFirst(repeated, 2)); // 1
        System.out.println(findLast(repeated, 2));  // 3
        System.out.println(countOccurrences(repeated, 2)); // 3
        
        int[] rotated = {4, 5, 6, 7, 0, 1, 2};
        System.out.println(searchRotated(rotated, 0)); // 4
    }
}
```

---

## 25. Merge Two Sorted Arrays

### Problem
Merge two sorted arrays into one sorted array.

### Solutions

```java
public class MergeSortedArrays {
    
    // Method 1: Using extra space - O(m+n) time, O(m+n) space
    public static int[] merge(int[] arr1, int[] arr2) {
        int[] result = new int[arr1.length + arr2.length];
        int i = 0, j = 0, k = 0;
        
        while (i < arr1.length && j < arr2.length) {
            if (arr1[i] <= arr2[j]) {
                result[k++] = arr1[i++];
            } else {
                result[k++] = arr2[j++];
            }
        }
        
        while (i < arr1.length) {
            result[k++] = arr1[i++];
        }
        
        while (j < arr2.length) {
            result[k++] = arr2[j++];
        }
        
        return result;
    }
    
    // Method 2: In-place merge (nums1 has enough space) - O(m+n) time, O(1) space
    public static void mergeInPlace(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1;      // Last element of nums1's data
        int j = n - 1;      // Last element of nums2
        int k = m + n - 1;  // Last position in nums1
        
        while (j >= 0) {
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[k--] = nums1[i--];
            } else {
                nums1[k--] = nums2[j--];
            }
        }
    }
    
    // Method 3: Using Streams
    public static int[] mergeStream(int[] arr1, int[] arr2) {
        return IntStream.concat(Arrays.stream(arr1), Arrays.stream(arr2))
            .sorted()
            .toArray();
    }
    
    // Method 4: Using PriorityQueue (for k sorted arrays)
    public static int[] mergeKArrays(int[][] arrays) {
        PriorityQueue<int[]> pq = new PriorityQueue<>(
            Comparator.comparingInt(a -> a[0])
        );
        
        int totalLength = 0;
        for (int i = 0; i < arrays.length; i++) {
            if (arrays[i].length > 0) {
                pq.offer(new int[]{arrays[i][0], i, 0}); // value, arrayIndex, elementIndex
                totalLength += arrays[i].length;
            }
        }
        
        int[] result = new int[totalLength];
        int idx = 0;
        
        while (!pq.isEmpty()) {
            int[] current = pq.poll();
            result[idx++] = current[0];
            
            int arrayIdx = current[1];
            int elementIdx = current[2];
            
            if (elementIdx + 1 < arrays[arrayIdx].length) {
                pq.offer(new int[]{
                    arrays[arrayIdx][elementIdx + 1],
                    arrayIdx,
                    elementIdx + 1
                });
            }
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        int[] arr1 = {1, 3, 5, 7};
        int[] arr2 = {2, 4, 6, 8};
        System.out.println(Arrays.toString(merge(arr1, arr2)));
        // [1, 2, 3, 4, 5, 6, 7, 8]
        
        // In-place merge
        int[] nums1 = {1, 2, 3, 0, 0, 0};
        int[] nums2 = {2, 5, 6};
        mergeInPlace(nums1, 3, nums2, 3);
        System.out.println(Arrays.toString(nums1));
        // [1, 2, 2, 3, 5, 6]
        
        // Merge K arrays
        int[][] kArrays = {{1, 4, 7}, {2, 5, 8}, {3, 6, 9}};
        System.out.println(Arrays.toString(mergeKArrays(kArrays)));
        // [1, 2, 3, 4, 5, 6, 7, 8, 9]
    }
}
```

---

# Quick Reference Summary

| # | Problem | Key Technique | Time | Space |
|---|---------|---------------|------|-------|
| 1 | Reverse String | Two Pointers | O(n) | O(n) |
| 2 | Palindrome Check | Two Pointers | O(n) | O(1) |
| 3 | Anagram Check | Character Count | O(n) | O(1) |
| 4 | First Non-Repeated | LinkedHashMap | O(n) | O(k) |
| 5 | Duplicate Characters | HashSet | O(n) | O(k) |
| 6 | Longest Substring | Sliding Window | O(n) | O(k) |
| 7 | Second Largest | Single Pass | O(n) | O(1) |
| 8 | Missing Number | Sum/XOR | O(n) | O(1) |
| 9 | Two Sum | HashMap | O(n) | O(n) |
| 10 | Move Zeroes | Two Pointers | O(n) | O(1) |
| 11 | Find Duplicate | Floyd's Cycle | O(n) | O(1) |
| 12 | Rotate Array | Reverse | O(n) | O(1) |
| 13 | Max Subarray | Kadane's | O(n) | O(1) |
| 14 | Prime Check | Trial Division | O(√n) | O(1) |
| 15 | Fibonacci | Iteration/Memo | O(n) | O(1)/O(n) |
| 16 | Reverse Number | Modulo | O(log n) | O(1) |
| 17 | Palindrome Number | Half Reverse | O(log n) | O(1) |
| 18 | GCD/LCM | Euclidean | O(log n) | O(1) |
| 19 | Reverse Linked List | Pointer Swap | O(n) | O(1) |
| 20 | Detect Cycle | Floyd's Tortoise | O(n) | O(1) |
| 21 | Valid Parentheses | Stack | O(n) | O(n) |
| 22 | Merge Sorted Lists | Two Pointers | O(n+m) | O(1) |
| 23 | Kth Largest | Min Heap | O(n log k) | O(k) |
| 24 | Binary Search | Divide & Conquer | O(log n) | O(1) |
| 25 | Merge Sorted Arrays | Two Pointers | O(n+m) | O(n+m) |

---

# Tips for Problem Solving Interviews

## Approach

1. **Understand the problem** - Ask clarifying questions
2. **Work through examples** - Use small inputs first
3. **Identify patterns** - What technique applies?
4. **Start with brute force** - Then optimize
5. **Code cleanly** - Variable names, comments
6. **Test your solution** - Edge cases, boundary conditions

## Common Patterns

| Pattern | Problems |
|---------|----------|
| **Two Pointers** | Palindrome, Two Sum (sorted), Reverse |
| **Sliding Window** | Longest Substring, Max Sum Subarray |
| **HashMap** | Two Sum, Anagram, Duplicate Detection |
| **Stack** | Valid Parentheses, Expression Evaluation |
| **Binary Search** | Sorted Array Search, Rotated Array |
| **Fast & Slow Pointers** | Cycle Detection, Middle of List |
| **Recursion + Memo** | Fibonacci, Tree Problems |
| **Kadane's Algorithm** | Maximum Subarray Sum |

## Edge Cases to Consider

- Empty input
- Single element
- All same elements
- Negative numbers
- Integer overflow
- Null values

---

*Good luck with your interviews!*
