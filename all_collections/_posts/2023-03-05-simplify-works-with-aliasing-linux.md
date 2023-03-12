---
layout: post
title: Đơn giản hóa, và Tối ưu Command Line bằng Aliasing trong Linux
date: 2023-03-03
categories: ["linux", "dev", "trick", "vi"]
---

Với những ai yêu thích sử dụng terminal và command line để thao tác công việc thường ngày thay vì sử dụng UI với nhứng cú click chuột _lách ta lách tách_ thì chắc hẳn sẽ yêu thích hệ điều hành Linux (Linux kernel). và dĩ nhiên mình cũng là một trong số đó 😊. TUY NHIÊN có khi nào bạn cảm thấy việc gõ đi gõ lại một command line dài dài, thêm vào đó là những option them kèm khó nhớ. Ví dụ như:

Gửi một HTTP request sử dụng CURL:

```bash
curl -verbose -X POST "http://localhost:xxxx" -Header "Content-Type: application/json" --data '{"name":"foo"}'
```

Hay là, in ra log của project có git với một format cụ thể:

```bash
git log --pretty=format:"%h - %an, %ar : %s"
```

Thành thật, dù có là người yêu thích _gõ phím_ hay sử dụng terminal đi chăng nữa thì chắc hẳn chúng ta cũng cảm thấy khá tốn thời gian, và phiền toái mỗi khi phải sử dụng những dòng lệnh dài như trên 🥲.

> Mình biết sẽ có bạn bảo có thể search lại những command đã dùng bằng `Ctrl + R`. Nhưng giả sử bạn dùng một command nhưng với option khác thì gần như cũng phải sửa lại thui à. Ví dụ như lệnh CURL bên trên:

> $ curl -verbose -X PUT "http://localhost:xxxx" -Header "Content-Type: application/json" --data '"{"age":"18"}"'

Trong bài viết này, mình sẽ chia sẻ cách mình sử dụng [Aliasing](https://www.bing.com/ck/a?!&&p=270da9eb087de49fJmltdHM9MTY3ODU3OTIwMCZpZ3VpZD0yNTA3YTg3NC1hM2NiLTY3NGQtMjc0OC1iYWU0YTI0ZDY2YTgmaW5zaWQ9NTIxOA&ptn=3&hsh=3&fclid=2507a874-a3cb-674d-2748-bae4a24d66a8&psq=alias+linux+wiki&u=a1aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvQWxpYXNfJTI4Y29tbWFuZCUyOQ&ntb=1), và `Function` trong ngôn ngữ lập trình [Bash Script](<https://www.wikiwand.com/en/Bash_(Unix_shell)>) để đơn giản hóa, và tối ưu các command line thường dùng ở môi trường Linux nhé.

## Prerequisites

- Linux (Ubuntu, Debian)
  - WSL, WSL2
- Biết dùng Command Line và Terminal cơ bản

## Getting Started

Trước khi bắt đầu, ta cần biết đôi chút về Aliasing.

> Trong Linux, alias là một shortcut, có thể dùng để gán vào một lệnh cụ thể giúp làm ngắn gọn, và đơn giản hóa lệnh đó. Nó giúp chúng ta có thể thoải mái tùy chỉnh và gán alias cho lệnh bất kì. khi sử dụng những lệnh được gán với alias, chúng ta chỉ cần gọi tên của alias đã được gán với lệnh.

Bước đầu tiên, mở `.bashrc` file để thao tác:

```
$ vim ~/.bashrc
```

Nếu bạn sử dụng [zsh]() thay vì `bashrc` (mình sử dụng zsh):

```
$ vim ~/.zshrc
```

### Aliasing

#### #Network

- In ra tất cả các [TCP](link) ports đang được active, sau đó in ra các thống kê liên quan đến chúng:

```bash
alias tcp_active='echo "> Here are TCP Listening ports:" && netstat -at && echo "\n> Statistics for all ports:" && netstat -st'
```

Sử dụng như sau, ra terminal bạn chỉ cần gõ tên của alias đã được gán:

```
$ tcp_active
```

Kết quả minh họa

![tcp active](https://user-images.githubusercontent.com/123849429/224535430-e6c4ba0c-78b4-4b92-820b-1f819887f64b.png)

- In ra tất cả các [UDP](link) ports đang được active, sau đó in ra các thống kê liên quan đến chúng:

```bash
alias udp_active='echo "> Here are UDP Listening ports:" && netstat -au && echo "\n> Statistics for all ports:" && netstat -su'
```

Sử dụng:

```
$ udp_active
```

Kết quả minh họa:

![image](https://user-images.githubusercontent.com/123849429/224535292-f7e26a21-7a18-45ad-8700-9ecc93662f5d.png)

- In ra tất cả các UNIX ports đang được active, sau đó in ra các thống kê liên quan đến chúng:

```bash
alias unix_active='echo "> Here are UNIX Listening ports:\n" && echo "\n> Statistics for all ports:" && netstat -lx'
```

- In ra tất cả các chương trình đang được active, sau đó in ra các thống kê liên quan đến chúng:

```bash
alias active_all='echo "> Here are all listening programs:\n" && netstat -lp'

# thay vì `netstat`, bạn có thể sử dụng `lsof`
alias active_port='sudo lsof -i -P -n'
```

#### #System, và Files

- In ra 30 files có kích thước lớn nhất ở directory hiện tại:

```bash
alias largest_files='du -h -x -s -- * | sort -r -h | head -30'
```

- In ra tất cả các user hiện tại trong hệ thống:

```bash
alias sys_users='cut -d: -f1 /etc/passwd | sort'
```

- Chỉ cần ấn `c` là xóa luôn cái màn hình terminal hiện tại:

```bash
alias c='clear'
```

#### #Git

- In ra lịch sử Git log của project với format `<commit_hash> <origin> <commit_message>` trên cùng một dòng:

```bash
alias glo='git log --oneline'
```

- In ra lịch sử Git log của project với format cụ thể:

```bash
alias glp='git log --pretty=format:"%h - %an, %ar : %s"'
```

- Tạo một Git branch mới, rồi nhảy qua branch mới tạo đó:

```bash
alias gb='git checkout -b'
```

### Làm việc với Function

#### #System, và Files

- Tìm process đang được active thông qua một port cụ thể, rồi kill nó:

```bash
kill_port() {
  local port=$1
  # tách PID (Process ID) từ process mà ta muốn kill
  local pid=$(lsof -t -i :${port})

  # nếu PID không rỗng
  if [ -n "$pid" ]; then
    echo "Process running on port $port with PID $pid. Killing process..."
    kill $pid

    # kiểm tra xem nó đã bị kill chưa
    sleep 1
    if kill -0 $pid >/dev/null 2>&1; then
      echo "ERROR: Failed to kill process with ID $pid."
      exit 1
    else
      echo "OK: Process with ID $pid was killed successfully."
    fi
  else
    echo "No process found running on port $port."
    exit 1
  fi
}
```

Cách dùng:

```bash
$ kill_port <port>
```

- In ra lịch sử sử dụng của một task cụ thể.

```bash
hg() {
  local task=$1

  if [ -n "$task" ]; then
    history | grep "$task";
  else
    echo "EXCEPTION: history of what?"
    echo "E.g. history vim"
    exit 1
  fi
}
```

Cách dùng:

```bash
$ hg <task>
```

Kết quả minh họa:

![image](https://user-images.githubusercontent.com/123849429/224536195-55820e2a-d5d1-4fca-8fd4-dc417b236911.png)

- Hiển thị nội dung của directory ở định dạng giống như những nhánh cây. (Giống _tree_ command nhưng nhìn gọn hơn về cấu trúc).

```bash
dir_tree() {
  find $1 -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
}
```

Cách dùng:

```bash
$ dir_tree
```

Kết quả minh họa:

![dir_tree](https://user-images.githubusercontent.com/123849429/224538095-209feb9e-8802-415d-8b73-4751d66f8fc0.png)

- Chuyển file từ host machine tới remote server sử dụng giao thức [SCP](https://www.wikiwand.com/en/Secure_copy_protocol).

```bash
# cài aliasing cho `scp`
alias scp_='scp'

# LÍ DO CẦN CÀI `aliasing` cho `scp` LÀ: bởi vì `scp` không phải là built-in
# (thứ có sẵn như những lệnh `cd`, `pwd`, hay `ls`...) nên trước khi dùng ta phải gán aliasing cho nó.

scp_do() {
  local file=$1
  local body=$2

  if [[ -n "$file" && -n "$host"  &&  -n "$path" ]]; then
    scp_ -v $file $body
  else
    echo "EXCEPTION: invalid arguments."
    echo "E.g. $ scp -v <file> <user>@<ip>:/destination_path"
    exit 1
  fi
}
```

Cách dùng:

```
$ scp_do -v <file> <user>@<ip> <destination_path>
```

- Thực thi gửi HTTP request sử dụng CURL

```bash
# cài aliasing cho `curl`
alias curl_='curl'

# lí do phải cài, mình có giải thích ở phần `scp` bên trên rồi nha.

curl_do() {
  local method=$1
  local url=$2
  local body=$3

  case $method in
    "GET") # nếu request là GET
      curl_ -v "http://$url" | json ;;

    "DELETE") # nếu request là DELETE
      curl_ -v -X $method "http://$url" | json ;;

    "PUT") # nếu request là PUT
      curl_ -v -X $method "http://$url" -H "Content-Type: application/json" -d $body | json ;;

    "POST") # nếu request là POST
      curl_ -v -X $method "http://$url" -H "Content-Type: application/json" -d $body | json ;;

    *)
      echo "Exception: invalid HTTP request method: $method"
      echo "E.g. curl_do <GET | DELETE | PUT | POST> <url> <body>"
      exit 1
      ;;
  esac
}
```

Cách dùng:

```bash
# GET
$ curl_do GET localhost:xxxx

# POST
$ curl_do POST localhost:xxxx '{"username": "foo", "password":"bar"}'

# PUT, DELETE tương tự nha...
```

- Tìm file hoặc directory ở một ví trí cụ thể bằng pattern, rồi xóa nó:

```bash
find_rm() {
  local location=$1
  local type=$2
  local pattern=$3

  case $type in
    "f") # nếu là file
        find $location -type $type -name "*$pattern*" -delete ;;

    "d") # nếu là directory
        find $location -type $type -name "*$pattern*" -delete ;;

    *) # không hợp lệ
      echo "ERROR: invalid type (only accept `f` or `d`)"
      echo "E.g. find_rm <location> <f | d> <pattern>"
      return 1
    ;;
  esac
}
```

Cách dùng:

```bash
# tìm file và xóa
$ find_rm <location_muốn_tìm> f <pattern>

# tìm directory và xóa
$ find_rm <location_muốn_tìm> d <pattern>
```

#### #Git

- Tạo git cho project:

```bash
git_init() {
  local dir=$1

  if [ -z "dir" ]; then
    echo "EXCEPTION: invalid arguments. Please provide a directory name.";
    echo "E.g. git_init test-dir"
    exit 1
  else
    mkdir $dir && cd $dir pwd git init
    touch README.md .gitignore LICENSE
  fi
}
```

- Kiểm tra xem hiện tại đang ở branch nào trong project:

```bash
git_br() {
    if [ -d .git ] ; then
        printf "%s" "> You are at branch: $(git branch 2> /dev/null | awk '/\*/{print $2}'))";
    fi
}
```

- Đẩy local branch lên remote origin:

```bash
git_po() {
  local origin=$1

  git push origin -u $origin
}
```

- Thêm Git url vào remote origin:

```bash
git_rao() {
  local url=$1

  git remote add origin $url
}
```

- Chỉ định initial/default branch cho repository:

```bash
git_br() {
  local br=$1

  git branch -M $br
}
```

- Viết commit

```bash
git_cm() {
  local msg=$1
  git commit -m $msg
}
```

- Xóa file ở Git remote repository nhưng vẫn giữ lại ở local:

```bash
git_rm_local() {
  local file=$1
  local msg=$2
  local br=$3

  # nếu file không rỗng, xóa nó ở remote (và vẫn giữ nó lại ở local)
  if [ -n "$file" ]; then
    git rm --cached $file
  else
    echo "EXCEPTION: invalid arguments. <file> should not be null."
    exit 1
  fi

  # nếu tham số `msg` không rỗng, viết commmit cho nó
  if [ -n "$msg" ]; then
    git commit -m $msg
  fi

  # nếu tham số `msg` và `br` không rỗng, đẩy nó lên remote origin `br`
  if [[ -n "$msg" && -n "$br" ]]; then
    git push origin $3
  fi
}
```

Cách dùng:

```bash
# chỉ xóa file, và không commit hay push lên remote
$ git_rm_local <file>

# chỉ xóa file, và commit, không push lên remote
$ git_rm_local <file> <message>

# xóa file, commit, và push lên remote
$ git_rm_local <file> <message> <origin_branch>
```

- Chắc hản có bạn sẽ nghĩ, từ đầu tới giờ, cũng kha khá nhiều lệnh rồi, sao mà nhớ hết, rồi cũng lại phải Google thôi...?? Hmm, ta lưu các lệnh trên vào một alias cụ thể, nếu quên hoặc không nhớ ta chỉ cần gọi alias đó ra để xem là được nha.

```bash
alias my_aliases='echo "
  $ tcp_active: Giải thích về lệnh này. Cách dùng ..... \n
  $ udp_active: Giải thích về lệnh này. Cách dùng ..... \n
  ....
  ....
  ....
" '
```

> LƯU Ý: Những thứ liên quan đến code, tên hàm, biến, tham số, cách gán, logic... tất cả được đặt theo quan điểm của mình, chứ không theo một [Naming convention](<https://www.wikiwand.com/en/Naming_convention_(programming)>), hay [Clean code](https://www.bing.com/search?q=clean+code+wiki&qs=n&form=QBRE&sp=-1&lq=0&pq=clean+code+wiki&sc=7-15&sk=&cvid=E7CAC7560B41405DA6F81ECB76D14595&ghsh=0&ghacc=0&ghpl=) cụ thể nào cả. Mình cố gắng trình bày đầy đủ nhất có thể cho người mới học có thẻ hiểu được. Bạn có thể copy về rồi trình bày theo cách của riêng bạn nha.

## Conclusion

Trên là những thứ mình tông hợp lại để phục vụ công việc thường ngày trong môi trường Linux. Nó giúp mình tối ưu, và tiết kiệp kha khá thời gian gõ phím và Google để tìm lại những lệnh cụ thể đã dùng. Hi vọng bạn tìm được điều gì đó hữu ích trong bài viết này. Chúc bạn học tập thành công.

**_Mình viết blog để tổng hợp lại những gì mình đã học, và cũng như học cách trình bày sao cho người khác có thể hiểu được, nên bài viết có thể tồn tại bug hoặc chưa hoàn thiện đâu đó. Nếu có gì liên quan đến bài viết, cần giúp debug, etc. thì đừng ngần ngại mà hãy cứ nhắn tin cho mình qua [Facebook](https://www.facebook.com/frankiie12a9/) nha._**
