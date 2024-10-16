---
Created: 2024-07-31T23:09
---
- Sort chars in string
    
    ```Go
    // Convert string arary of strings splitting it by chars; 
    // use sort.Strings to sort the slice of strings; join slice into string.
    import (
    	"strings"
    	"sort"
    )
    
    func sortStr(word string) string {
    	sortedSlice := strings.Split(word, "")
    	sort.Strings(sortedSlice)
    	return strings.Join(sortedSlice)
    }
    
    // Convert string to slice of runes; sort them with sort func;
    // join slice into string.
    
    func sortStr(word string) string {
    	sortedSlice := []rune(word)
    	sort.Sort(sortedSlice, func(i, j int) bool {
    		return sortedSlice[i] < sortedSlice[j]
    	}
    	return string(sortedSlice)
    }
    ```
    
- Make Go available in the `shell`
    - Go `cli` tool is usually installed into `/usr/local/go/bin` directory. To make it globally available in the `shell` append `export PATH=$PATH:/usr/local/go/bin`to your `$HOME/.bashrc` file.