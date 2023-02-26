---
title: Hướng dẫn setup ssl config để làm việc cùng lúc với nhiều tải khoản Github
layout: post
date: 2023-02-27
categories: ["git", "github", "dev"]
---

<!-- ---

layout: post
title: "Kinh nghiệm tạo website cá nhân với Jekyll"
subtitle: "Một số kinh nghiệm tôi thu được sau khi chuyển website cá nhân từ Wordpress sang Jekyll"
date: 2016-09-16
categories: [Jekyll]
tags: [Jekyll, website]
permalink: /blogging/kinh-nghiem-tao-webiste-ca-nhan-voi-jekyll/
bigimg: "/assets/img/blogging/jekyll/jekyllhomepage.png"

--- -->

# Hướng dẫn setup SSL config để làm việc cùng lúc với nhiều tải khoản Github

GitHub provides two ways of connecting to git repositories, namely SSH and HTTPS. HTTPS requires you to supply an access token every time you push to a repository. SSH allows you to push code without remembering your username and token every time you push code to a GitHub repository.

So you have a personal GitHub account—everything is working perfectly. But then, you get a new job, and you now need to be able to push and pull to multiple accounts. How do you do that? I'll show you how!

### Create a New SSH Key

<!-- We need to generate a unique SSH key for our second GitHub account. -->

Trước tiên, chúng ta cần phải tạo SSH key cho tài khoản Github thứ hai mà ta muốn dùng. Ở vị trí terminal hiện tại, chúng ta chuyển hướng tới `.ssh` folder và tạo key thông qua câu lệnh sau đây.

```bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096 -C "email-mà-bạn-đăng-kí-cho-tài-khoản-github"
```

Sau khi câu lệnh trên được thực thi thành công, sẽ có một cái prompt xuất hiện để bạn đặt tên file chứa SSH key.

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/cudayanh/.ssh/id_rsa): id_rsa_forwork
```

Bạn có thể điền tên file vào chỗ trống trên và ấn enter, mình điền là `id_rsa_forwork`.

Tiếp đến sẽ là hai cái prompt yêu cầu điền **passphrase** cho file vừa khởi tạo, mục này không bắt buộc nên các bạn có thể để trống và ấn enter hai lần là xong.

```bash
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

<!-- Once you enter the passphrase again, the key is saved in the default location you specified, and two files are created as shown below. -->

Oki, sau khi nhập passphrase xong, SSH key sẽ được lưu ở hai file riêng biệt, **identification** tức private key sẽ được lưu ở `id_rsa_forwork` còn **public key** thì lưu ở `id_rsa_forwork.pub` như kết quả ở dưới đây.

```bash
Your identification has been saved in id_rsa_forwork
Your public key has been saved in id_rsa_for.pub
The key fingerprint is:
SHA256:1asgVJYdT+QMU3izczOA109WlwSpynb7sq2ntejX7Go newstufflover@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
|        oo+*+o+.=|
|       o. +O=...+|
|      .   .o*+ + |
|     .   . .o.+ .|
|      . S . .o o |
|       . = o     |
|        . o ..o  |
|           o+E.o |
|          .*X=o. |
+----[SHA256]-----+
```

Giờ cái chúng ta cần copy cái public key lưu ở file`id_rsa_forwork.pub` để có thể kết nối với tài khoản Github. Thực thi câu lệnh sau chúng ta có thể thấy được nội dung (public key) trong file.

```
$ cat id_rsa_forwork.pub
```

Copy nó rồi lưu tạm vào đâu trên máy nhé.

### Attach the New Key

Oki, chúng ta cần truy cập vào github và đăng nhập tài khoản, tiếp đó ta click vào mục **Settings**.

![setting profile](https://user-images.githubusercontent.com/123849429/220717258-e78be23b-44b6-4d91-be51-924ae7a8b86c.png)

Rồi click tiếp vào mục **SSH and GPG keys**.

![SSH and GPG keys](https://user-images.githubusercontent.com/123849429/220717609-720b3619-1fab-4ad4-89d4-d21bbbb8f0b2.png)

Sau đó, click vào cái nút **New SSH key** để điền thông tin key.

![generate new key](https://user-images.githubusercontent.com/123849429/220718572-aa2a9cfd-291d-4303-b52e-80ed77bb4257.png)

ở phần **Title** chúng ta có thể điền tùy ý cũng oke nha. Còn ở phần **Key** thì paste cái public key mà chúng ta lưu trước đó ở mục trên vào là xong.

![add new key](https://user-images.githubusercontent.com/123849429/220720055-3c72f9b4-1563-4d19-9153-6d1a5a07bd74.png)

### Add the SSH Key to the Agent

Chúng ta đã tạo SSH key (public và private) tiếp đó là đã thêm nó vào tài khoản Github mà chúng ta muốn dùng đúng không nào. Tuy nhiên như thế vẫn chưa xong, để SSH có thể nhận diện ra được là SSH key (được lưu ở `id_rsa_forwork`) và tài khoản Github đã được kết nối với nhau thông qua một cái file (`id_rsa_forwork.pub`) cụ thể nào đó. Ta cần phải cho SSH biết điều đó bằng cách thêm nó vào danh sách identity của SSH.

```bash
$ ssh-add ~/.ssh/id_rsa_forwork
```

**_Lưu ý_**: Nếu sau khi thực thi câu lệnh trên mà bị lỗi sau đây, thì có thể là `ssh-agent` của bạn chưa được kích hoạt.

> Could not open a connection to your authentication agent

Để kích hoạt `ssh-agent`, ta chỉ cần thực thi câu lệnh sau đây.

```
$ eval `ssh-agent -s`

```

Và thực hiện lại câu lệnh thêm vào identity.

```
$ ssh-add ~/.ssh/id_rsa_forwork
```

Nếu không có vấn đề gì xảy ra thì ta sẽ nhận được kết quả thông báo identity đã được thêm thành công như dưới đây nha.

```bash
Identity added: id_rsa_forwork (newstufflover@gmail.com)
```

### Create a Config File

<!-- We've done the bulk of the workload, but now we need a way to specify when we wish to push to our personal account and when we should instead push to our company account. To do so, let's create a config file. -->

Sắpp xonggg rồiiii. Giờ ta đang có hai tài khoản Github, một cái đang dùng, giờ ta muốn thêm một tài khoản khác để dùng song song mà không bị trùng lặp nhau, điều đó cũng có nghĩa là ta phải tạo riêng Github config cho mỗi tài khoản đúng không nào. Oki, giờ ta vào file SSH config bằng câu lệnh sau.

```bash
$ vim ~/.ssh/config
```

Sau khi mở file `~/.ssh/config`, chúng ta có thể sẽ thấy một vài cái config cho tài khoản github. (tùy từng người có thể sẽ không giống nhau nha)

```bash
  # giả sử cái này là config cho tài khoản github trước đó
  Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_forpersonal

  # thì cái này sẽ là config cho tài khoản bây giờ
  Host github.com-forwork    # để ý dòng này nha
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_forwork  # dòng nàyyy nữa nha
```

Để có thể dễ dàng phân biệt đâu là config của tài khoản trước đó, và đâu là config của tài khoản bây giờ, ta sẽ
thay đổi cái **Host** của tài khoản bây giờ thành **github.com-forwork** nha.

<!-- This time, rather than setting the host to github.com, we've named it github.com-work. The difference is that we're now attaching the new identity file that we created previously: `id_rsa_forwork`. -->

Xong việc rồi thì lưu và thoát ra nào. Đối với những bạn nào chưa dùng **vim** bao giờ thì có thể ấn phím lần lượt theo dưới đây nha.

- ấn `esc` rồi tiếp đó ấn `:` rồi ấn `wq`.

### Try It Out!

<!-- It's time to see if our efforts were successful. Create a `test-repo` directory, and change directory to it, then add a `README.md` file. -->

Giờ chính thức xong rồi này. Xong thì xong nhưng cũng phải test cái xem có chạy được hay không chứ nhỉ 😊.

Ta cần tạo một cái directory để chứa project.

```bash
$ cd ~/
$ mkdir test-repo
$ cd test-repo
$ touch README.md
```

Giờ ta vào Github để tạo một cái remote project (nhớ là đăng nhập bằng tài khoản chúng ta vừa dùng để config nha)

![init repo](https://user-images.githubusercontent.com/123849429/220728028-47a321d4-3e8f-4caf-81dd-bf78d3547618.png)

Ta khởi tạo local repository cho project.

```
$ git init
$ git add .
$ git commit -m "initial commit"
```

Chỗ này _khoan_ một chút nha. Còn nhớ cái **Host** trên `~/.ssh/config` mà chúng ta vừa config cho tài khoản Github hiện giờ chứ? Đúng nó rồi đấy. Lúc push project lên remote, ta sẽ sử dụng Host là **github.com-forwork** chứ không còn là **github.com** như trước nữa (vì nó đã thuộc về tài khoản trước đó roài)

```
$ git remote add origin git@github.com-forwork:codeforfut17/test-add-mutiple-github-account.git
$ git push origin main
```

Pangggggg..... Done nha!

```bash
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Writing objects: 100% (4/4), 263 bytes | 263.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com-forwork:codeforfut17/test-add-mutiple-github-account.git
 * [new branch]      main -> main

```

Giờ thì lên tài khoản Github vừa rồi ấn F5 và kiểm tra thôi 😊.

### Conclusion

Thông qua bài viết này, chúng ta đã cùng nhau tạo một config riêng cho tài khoản Github phụ để có thể làm việc riêng lẻ mà không bị trùng lặp với tài khoản đang dùng trên cùng một máy. Bài có hơi dài nhưng nếu bạn followed đến đoạn kết thì mình cảm ơn nha 😊.
