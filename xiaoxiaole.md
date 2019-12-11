```go
package main

import (
	"fmt"
	"math/rand"
	"strconv"
)

var (
	row   = 8
	col   = 8
	board [8*8 + 1]string
	icon  map[int]string
)

func main() {
	boardInit()

	for 1 == 1 {
		var postion1, postion2 int
		// fmt.Scanln(&postion1, postion2)
		fmt.Println("请输入交换的两个棋子坐标：")
		fmt.Scanln(&postion1)
		fmt.Scanln(&postion2)
		move(postion1, postion2)
	}
}

//棋盘初始化
func boardInit() {
	for i := 0; i < row; i++ {
		for j := 1; j <= col; j++ {
			board[i*col+j] = strconv.Itoa(rand.Intn(5) + 1)
		}
	}
	res := findDestroyAbleCubes()
	for len(res) != 0 {
		for i := 0; i < len(res); i++ {
			board[res[i]] = strconv.Itoa(rand.Intn(5) + 1)
		}
		res = findDestroyAbleCubes()
	}
	print()
}

//移动
func move(postion1, postion2 int) {
	swap(postion1, postion2)
	res := findDestroyAbleCubes()
	if len(res) == 0 {
		fmt.Println("该移动无效")
		swap(postion2, postion1)
	} else {
		destroy(res)
	}
}
func swap(x, y int) {
	tmp := board[x]
	board[x] = board[y]
	board[y] = tmp
}

//查找可消除的元素块
func findDestroyAbleCubes() []int {
	res := make([]int, 0)
	var countCol = 1
	var countRow = 1
	//横向
	for i := 0; i < row; i++ {
		for j := 2; j <= col; j++ {
			if board[i*col+j] == board[i*col+(j-1)] {
				countCol++
			} else {
				if countCol >= 3 {
					for t := j - countCol; t <= j-1; t++ {
						res = append(res, i*col+t)
					}
				}
				countCol = 1
			}
		}
		countCol = 1
	}
	// 纵向
	for j := 1; j <= col; j++ {
		for i := 1; i < row; i++ {
			if board[i*col+j] == board[(i-1)*col+j] {
				countRow++
			} else {
				if countRow >= 3 {
					for t := i - countRow; t <= i-1; t++ {
						res = append(res, t*col+j)
					}
				}
				countRow = 1
			}
		}
	}
	return res
}

//推荐
func prompt() {
	var countCol = 1
	var countRow = 1
	//横向
	for i := 0; i < row; i++ {
		for j := 2; j <= col; j++ {
			if board[i*col+j] == board[i*col+(j-1)] {
				countCol++
			} else {
				if countCol >= 2 {

				}
				countCol = 1
			}
		}
		countCol = 1
	}
	// 纵向
	for j := 1; j <= col; j++ {
		for i := 1; i < row; i++ {
			if board[i*col+j] == board[(i-1)*col+j] {
				countRow++
			} else {
				if countRow >= 2 {

				}
				countRow = 1
			}
		}
	}
}

//消除，用*替代
func destroy(res []int) {
	for i := 0; i < len(res); i++ {
		board[res[i]] = "*"
	}
	print()
	decline(res)
}

//下降   这里随机替换模拟
func decline(res []int) {
	for i := 0; i < len(res); i++ {
		board[res[i]] = strconv.Itoa(rand.Intn(5) + 1)
	}
	print()
}

//打印棋盘
func print() {
	for i := 0; i < row; i++ {
		for j := 1; j <= col; j++ {
			fmt.Print(board[i*col+j] + "  ")
		}
		fmt.Println()
	}
	fmt.Println()
}
```