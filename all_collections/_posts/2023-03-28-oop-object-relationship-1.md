b---
layout: post
title: OOP 101 - Object Relationships (phần 1)
date: 2023-03-28
categories: ["dev", "oop", "cplusplus", "vi"]

---

Lập trình hướng đối tượng được xây dựng trên ý tưởng về các đối tượng và mối quan hệ của chúng với nhau. Những mối quan hệ này rất cần thiết trong việc tạo ra các chương trình có tính mô-đun, có thể bảo trì và hiệu quả. Việc hiểu các loại quan hệ khác nhau giữa các đối tượng là rất quan trọng để thiết kế các hệ thống phần mềm hiệu quả.

Mối quan hệ đối tượng (Object Relationship) đề cập đến cách các đối tượng được liên kết và tương tác với nhau trong một hệ thống. Trong lập trình hướng đối tượng, chúng ta có thể liệt kê ra một số loại quan hệ đối tượng thường được dùng như, `kế thừa (Object Inheritance)`, `thành phần (Ọbject Composition)`, `liên kết (Object Association)`, `tập hợp (Object Aggregation)`, `phụ thuộc (Object Dependency)` và `thùng chứa (Container Class)`. Mỗi một loại quan hệ trên được dùng để minh họa, hay biểu diễn cách mà các đối tượng liên kết, và tương tác với nhau.

Trong bài viết này, chúng ta sẽ cùng tìm hiểu và so sánh các loại quan hệ đối tượng là `Composition`, `Aggregation`, và `Association`.

### 1. Object Composition

**Object Composition** là một khái niệm dùng để biểu diễn quan hệ giữa một object lớn được tạo thành bởi chính nó kết hợp với nhiều object nhỏ và đơn giản hơn gộp lại. Chúng ta có thể dễ dàng thấy được khái niệm này thông qua nhiều minh họa thực tế trong cuộc sống. Ví dụ như **chiếc máy tính**, **xe ô tô**,... Như ta đã biết, để có thể tạo ra được một chiếc máy tính hoàn chỉnh, chúng ta phải cần đến rất nhiều thành phần, hay bộ phận như, `CPU`, `bộ lưu trữ (Storage)`, và `thiết bị I/O (Display)`, hay để tạo nên chiếc ô tô, chúng ta cần `thân xe`, `bánh xe`, `động cơ`, và nhiều bộ phận đơn lẻ khác nữa.

Composition là một loại quan hệ "has-a", trong đó một đối tượng sở hữu một đối tượng khác và vòng đời của đối tượng sở hữu gắn liền với vòng đời của đối tượng bị sở hữu. Nói cách khác, _khi vòng đời của đối tượng sở hữu bị kết thúc, thì đồng nghĩa, vòng đời của đối tượng mà nó sở hữu cũng sẽ bị kết thúc theo_. Ví dụ, một chiếc máy tính "có-một" cái CPU, và khi máy tính đó hỏng, thì CPU của nó cũng bị hỏng. ĐÓ CHÍNH LÀ NGUYÊN LÝ CỦA Object Composition.

> Object Composition gồm có hai phân nhóm chính, Composition và Aggregation.

Những đối tượng được coi là đủ điều kiện của một Object Composition, thì chúng cần phải có những yếu tố sau đây:

- đối tượng (bị sở hữu) là một phần của đối tượng (sở hữu)
- đối tượng (bị sở hữu) chỉ có thể thuộc về một đối tượng (sở hữu) tại một thời điểm
- đối tượng (bị sở hữu) có sự tồn tại của nó được quản lý bởi đối tượng (sở hữu)
- đối tượng (bị sở hữu) không biết về sự tồn tại của đối tượng (sở hữu)

#### Ví dụ Code

Chúng ta sẽ lấy vị dụ `máy tính (Computer)` và những thành phần bên trong của nó như `CPU`, `bộ nhớ (Storage)`, và `(Display)`.

```
+----------------------------+
|          Computer          |
|----------------------------|
|           CPU              |
|          Storage           |
|          Display           |
+----------------------------+
   |          |          |
   |          |          |
+------+ +---------+ +---------+
| CPU  | | Storage | | Display |
+------+ +---------+ +---------+
```

Mỗi một máy tính thì đều cần phải có CPU để xử lí những tính toán, bộ nhớ để lưu trữ dữ liệu, và Display để hiển thị kết quả.

`Components.h`

```cpp
#ifndef COMPONENTS_H
#define COMPONENTS_H

#include <iostream>
#include <string>

struct CPU
{
  int core_count;
  int clock_speed;
};

struct Storage
{
  int size;
  std::string type;
};

struct Display
{
  int resolution;
  int refresh_rate;
};

#endif
```

`Computer.h`

```cpp
#ifndef COMPUTER_H
#define COMPUTER_H

#include <iostream>
#include "Components.h"

class Computer
{
private:
  std::string m_name;

  // Composition - a Computer sở hữu CPU, Storage, và Display
  CPU m_cpu;
  Storage m_storage;
  Display m_display;

public:
  Computer(
    const std::string &name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : m_name{ name }, m_cpu{ CPU }, m_storage{ storage }, m_display{ display } {}

  // Overloaded output operator
  friend std::ostream& operator<<(std::ostream& out, const Computer& computer) {
    out << computer.m_name << std::endl;
    out << "CPU (" << "clock_speed: " << computer.m_cpu.clock_speed << ", core_cout: " << computer.m_cpu.core_count << ")" <<  std::endl;
    out << "Storage (" << "size: " << computer.m_storage.size << ", type: " << computer.m_storage.type << ")" << std::endl;
    out << "Display (" << "resolution: " << computer.m_display.resolution << ", refresh_rate: " << computer.m_display.refresh_rate << ")" << std::endl;
    return out;
  }
}

// Driver
int main() {
  {
    // vòng đời của CPU, Storage, và Display đều bị ràng buộc bởi Computer
    // một khi vòng đời của Computer kết thúc, chúng cũng sẽ bị kết thúc theo
    Computer computer {
      "Dell DX-3",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1920 * 1080, 30} // Display
    };

    std::cout << computer << std::endl;
  }

  return 0;
}

#endif
```

### 2. Object Aggregation

Aggregation cũng là một kiểu quan hệ "has-a" giống Composition, một đối tượng sở hữu hay hợp tác với một đối tượng khác. Tuy nhiên, so với Composition, Aggregation khác ở chỗ, hai đối tượng sẽ không bị ràng buộc bởi đối phương về vòng đời và sự hiện diện của chúng. Nói một cách khác, chúng có thể kết hợp với nhau để thực hiện một công việc, sau khi hoàn thành, chúng có thể tách rời nhau. Nếu vòng đời của một đối tượng bất kỳ bị kết thúc thì nó sẽ không ảnh hưởng tới vòng đời của đối tượng còn lại.

Chúng ta có thể minh hoạ Object Aggregation thông qua ví dụ như, `chiếc máy tính (Computer)` và bộ phận I/O như `chuột (Mouse)`, và `bàn phím (Keyboard)`. Mỗi máy tính có thể tiếp nhận một hoặc nhiều con chuột, và bàn phím để thao tác công việc nhận đầu vào từ bên ngoài. Việc tồn tại của máy tính thì cũng không ảnh hưởng tới chuột, và bàn phím, chúng có thể tồn tại độc lập lẫn nhau, cũng như việc máy tính kết nối với loại chuột, hay bàn phím nào thì cũng không ảnh hưởng tới chức năng bên trong của nó và ngược lại.

Đối tượng được coi là đủ điều kiện của một Object Aggregation, thì chúng cần phải có những yếu tố sau đây:

- Đối tượng (bị sở hữu) là một phần của đối tượng (sở hữu) trong một khoảng thời gian nhất định
- Sự hiện diện, tồn tại của đối tượng (bị sở hữu) không trực tiếp chịu sự quản lí hay bị ảnh hưởng bởi đối tượng (sở hữu), và ngược lại
- Đối tượng (bị sở hữu) không biết gì về sự tồn tại của đối tượng (sở hữu)

#### Code Ví dụ

Chúng ta sẽ lấy ví dụ về `máy tính (Computer)` liên kết với `chuột (Mouse)`, và `bàn phím (Keyboard)`.

```
                    +--------------------------+
                    |          Computer        |
                    |--------------------------|
                    |           Mouse          |
                    |          Keyboard        |
                    +--------------------------+
                          |              |
                          |              |
                      +-------+      +----------+
                      | Mouse |      | Keyboard |
                      +-------+      +----------+
```

> `Object Composition`, và `Aggregation` đều là kiểu quan hệ "has-a" nên cách biểu thị diagram cuả chúng cũng tương tự nhau.

Một cái máy tính có thể kết nối với một hoặc nhiều cái chuột, và một hoặc nhiều cái bàn phím.

`Peripheral.h`

```cpp
#ifndef PERIPHERAL_H
#define PERIPHERAL_H

struct Mouse
{
  std::string name;
  std::string brand;
};

struct Keyboard
{
  std::string name;
  std::string brand;
};

#endif
```

`Aggregation/Computer.h`

```cpp
#ifndef COMPUTER_H
#define COMPUTER_H

#include <iostream>
#include <string>
#include "Peripheral.h"

class Computer
{
private:
    std::string m_name;
    CPU m_cpu;
    Storage m_storage;
    Display m_display;

    // Aggregation - Computer có tham chiếu đến Mouse, và Keyboards chứ không sở hữu chúng
    Mouse m_mouse;
    std::vector<Keyboard> m_keyboards;

public:
    Computer(
      const std::string &name,
      const CPU &cpu,
      const Storage &storage,
      const Display &display
    )
      : m_name{ name }, m_cpu{ cpu }, m_storage{ storage }, m_display{ display }
    {}

    // Kết nối với một chuột
    void setMouse(Mouse mouse) {
      m_mouse = mouse;
    }

    // Kết nối với một hoặc nhiều bàn phím
    void addKeyboard(const Keyboard &keyboard) {
      m_keyboards.push_back(keyboard);
    }
};

// Driver
int main() {
    // Keyboard và Mouse được định nghĩa và khai báo độc lập với Computer
    Keyboard kb { "Keychron k4", "Keychron" };
    Mouse ms { "Logitech F12", "Logitech" };

    {
      Computer computer {
        "Dell DX-3",
        {8, 4},           // CPU
        {16, "SSD"},      // Storage
        {1920 * 1080, 50} // Display
      };

      // Liên kết Computer với Mouse và Keyboards,
      // vòng đời của Mouse, và Keyboards không bị ràng buộc bởi Computer
      computer.setMouse(ms);
      computer.addKeyboard(kb);

      std::cout << computer << std::endl;
    }

    return 0;
}

#endif
```

### 4. Object Association

Association là một kiểu quan hệ "uses-a", kiểu quan hệ này ám chỉ một đối tượng liên kết hay được liên kết với một đối tượng khác. Đối tượng liên kết và được liên kết tồn tại độc lập, riêng biệt lẫn nhau. Ta có thể thấy được kiểu quan hệ này được minh họa thông qua một vài ví dụ như, `một người dùng` sử dụng một `cái máy tính công cộng`, hay `một sinh viên` sử dụng một `quyển sách` trong thư viện\_... Sự tồn tại của máy tính/quyển sách độc lập với sự tồn tại của người dùng/sinh viên, và người dùng/sinh viên không trực tiếp sở hữu và quản lí vòng đời của máy tính/quyển sách.

Không giống như Composition hay Aggregation, nơi mà đối tượng sở hữu được tạo thành bởi sự kết hợp giữa chính nó và một hay nhiều đối tượng bị sở hữu. Đối với Association, các đối tượng liên kết/được liên kết với nhau nhưng lại không có liên quan đến nhau, chúng tồn tại độc lập, và riêng biệt lẫn nhau. Một đối tượng có thể liên kết với đối tượng này, và cũng đồng thời liên kết với một hoặc nhiều đối tượng khác. Thêm nữa, sự liên kết giữa các đối tượng có thể là `trực tiếp (directional)`, hoặc `gián tiếp (indirectional)`, và `đơn hướng (unidirectional)`, hoặc `đa hướng (bidirectional)`.

- `Liên kết trực tiếp (Directional association)` - hai đối tượng liên kết trực tiếp với nhau
- `Liên kết gián tiếp (Indirectional association)` - hai đối tượng liên kết với nhau thông qua một đối tượng khác trung gian
- `Liên kết đơn hướng (Unidirectional association)` - một liên kết mà trong đó chỉ có một đối tượng biết về mối quan hệ
- `Liên kết đa hướng (Bidirectional association)` - một liên kết mà cả hai đối tượng tham gia đều biết về mối quan hệ

Đối tượng để có thể được coi là một association, chúng phải có đủ những yếu tố sau đây:

- Đối tượng (được liên kết) không có liên quan, và tồn tại độc lập với đối tượng (liên kết)
- Đối tượng (được liên kết) có thể cùng một lúc được liên kết với nhiều đối tượng (liên kết), và ngược lại
- Đối tượng (được liên kết) không bị ràng buộc bởi vòng đời, hay sự tồn tại của đối tượng (liên kết)

#### Directional association

Dưới đây ta có ví dụ `người dùng (User)` và `máy tính (Computer)`

```
                      +------+     +----------+
                      | User |-----| Computer |
                      +------+     +----------+
```

Một đối tượng người dùng liên kết và sử dụng chiếc máy tính.

`User.h`

```cpp
#ifndef USER_H
#define USER_H

#include <string>

class User
{
  private:
    int m_id;
    std::string m_name;

  public:
    User(int id, std::string name) : m_id{ id }, m_name{ name } {}
};

#endif
```

`Association/Computer.h`

```c++
#ifndef COMPUTER_H
#define COMPUTER_H

#include <iostream>
#include <vector>
#include <string>
#include "User.h"

class Computer
{
private:
    std::string m_name;
    CPU m_cpu;
    Storage m_storage;
    Display m_display;
    Mouse m_mouse;
    std::vector<Keyboard> m_keyboards;

    // Association - Computer has a reference to a User, but doesn't own it
    User *m_user;

public:
    Computer(
      const std::string &name,
      const CPU &CPU,
      const Storage &storage,
      const Display &display
    )
      : m_name{ name }, m_cpu{ CPU }, m_storage{ storage }, m_display{ display }
    {}

    // Kết nối với một chuột
    void setMouse(Mouse mouse) {
        m_mouse = mouse;
    }

    // Kết nối với một hoặc nhiều bàn phím
    void addKeyboard(const Keyboard &keyboard) {
        m_keyboards.push_back(keyboard);
    }

    // Chỉ định liên kết Computer với User
    void setUsedBy(User *user) {
        m_user = user;
    }

    friend std::ostream& operator<<(std::ostream& out, const Computer& computer)
    {
      out << computer.m_name << std::endl;
      out << "CPU (" << "clock_speed: " << computer.m_cpu.clock_speed << ", core_cout: " << computer.m_cpu.core_count << ")" <<  std::endl;
      out << "Storage (" << "size: " << computer.m_storage.size << ", type: " << computer.m_storage.type << ")" << std::endl;
      out << "Display (" << "resolution: " << computer.m_display.resolution << ", refresh_rate: " << computer.m_display.refresh_rate << ")" << std::endl;
      out << "Mouse (" << "name: " << computer.m_mouse.name << "brand: " << computer.m_mouse.brand << ")"<< std::endl;
      for (int i = 0; i < computer.m_keyboards.size(); ++i) {
        out << "Keyboard " << (i + 1) << " (" << "name: " << computer.m_keyboards.at(i).name
          << ", brand: " << computer.m_keyboards.at(i).brand << ")" << std::endl;
      }
      if (computer.m_user != nullptr) {
        out << "Used by (" << "id: "<< computer.m_user->m_id << ", name: " << computer.m_user->m_name << ")" << std::endl;
      }
      return out;
    }
};

// Driver
int main() {
    Computer computer{
      "Dell DX-3",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1920 * 1080, 50} // Display
    };

    Keyboard kb{ "Keychron k4", "Keychron" };
    Keyboard kb1{ "Apple", "Magic keyboard" };
    Mouse ms{ "Logitech F12", "Logitech" };

    computer.setMouse(ms);
    computer.addKeyboard(kb);
    computer.addKeyboard(kb1);

    User user{ 1, "Alice" };

    // Gán User hiện tại cho Computer
    computer.setUsedBy(&user);

    std::cout << computer << std::endl;

    return 0;
}

#endif
```

#### Indirectional association

Khác với liên kết trực tiếp (directional association) được minh họa ở ví dụ trên thì liên kết gián tiếp (indirectional association) ám chỉ hai hay nhiều đối tượng liên kết với nhau thông qua một đối tượng khác làm trung gian. Ta có thể minh họa kiểu quan hệ này thông qua ví dụ một `người dùng (User)` kết nối với `một máy tính từ xa (Remote Computer)` bằng cách gián tiếp khi sử dụng một `máy tính trung gian (Computer)`.

<h6>Diagram minh họa</h6>

```
                +------+     +----------+     +-----------------+
                | User |-----| Computer |-----| Remote Computer |
                +------+     +----------+     +-----------------+
```

<h6>Code ví dụ:</h6>

```c++
#ifndef COMPUTER_H
#define COMPUTER_H

#include <iostream>
#include "User.h"

class RemoteComputer
{
private:
  std::string m_name;

  // other members are ommited for brevity

public:
  RemoteComputer(const std::string &name) : m_name{ name } {}
};

class Computer
{
private:
    std::string m_name;
    User *m_user;

    // Indirect asociation - User có thể gián tiếp kết nối với RemoteComputer thông qua Computer
    RemoteComputer *m_remoteComp;

public:
    Computer(const std::string &name) : m_name{ name } {}

    void setUsedBy(User *user) {
      m_user = user;
    }

    // Kết nối tới Remote Computer
    void accessRemoteComputer(RemoteComputer *remoteComp) {
      m_remoteComp = remoteComp;
    }

    friend std::ostream& operator<<(std::ostream& out, const Computer& computer)
    {
      out << computer.m_name << std::endl;
      if (computer.m_user != nullptr) {
        out << "Used by (" << "id: "<< computer.m_user->m_id << ", name: " << computer.m_user->m_name << ")" << std::endl;
      }
      return out;
    }
};

// Driver
int main() {
    Computer computer { "Dell DX-3" };

    RemoteComputer remoteComputer { "Macbook Pro" };

    User user {1, "alice"};

    computer.setUsedBy(&user);

    // User hiện tại kết nối với Remote Computer thông qua Computer trung gian
    computer.accessRemoteComputer(&remoteComputer);

    std::cout << computer << std::endl;

    return 0;
}
#endif
```

#### Unidirectional association

Dưới đây là ví dụ minh họa cho Liên kết đơn hướng. Đối tượng User chứa 0 hoặc nhiều Computer, Còn Computer thì không chứa bất kì một User nào cả. Điều này có nghĩa, từ User, chúng ta có thể gọi và sử dụng những thuộc tính (attributes) của Computer, tuy nhiên từ Computer, chúng ta không thể gọi và sử dụng bất kì thuộc tính nào của User.

<h6>Diagram minh họa</h6>

```
                      +------+       +----------+
                      | User |X----->| Computer |
                      +------+       +----------+
```

<h6>Code ví dụ:</h6>

```c++
#include <iostream>
#include <vector>
#include <string>

// Computer does't know anything about the User
class Computer
{
private:
    std::string m_name;

public:
    Computer(std::string name) : m_name{ name } {}
};

// User does know well about the Computer(s)
class User
{
private:
    std::string m_name;
    std::vector<Computer*> m_computers;

public:
    User(std::string name) : m_name{ name } {}

    void useComputer(Computer *computer) {
        m_computers.push_back(computer);
    }
};

// Driver
int main()  {
    User user{ "Alice" };
    Computer computer{ "Dell" };
    Computer computer1{ "HP" };

    user.useComputer(&computer);
    user.useComputer(&computer1);

    return 0;
}
```

#### Bidirectional association

Dưới đây là ví dụ minh họa cho Liên kết đa hướng. Đối tượng User chứa 0 hoặc nhiều Computer, Còn mỗi Computer thì sẽ chứa một User. Điều này có nghĩa, từ User, chúng ta có thể gọi và sử dụng những thuộc tính (attributes) của Computer, và từ Computer, chúng ta cũng có thể gọi hoặc sử dụng bất kì thuộc tính nào của User mà nó đang được liên kết.

<h6>Diagram minh họa</h6>

```
                          +------+        +----------+
                          | User |<------>| Computer |
                          +------+        +----------+
```

<h6>Code ví dụ:</h6>

```c++
#include <iostream>
#include <vector>
#include <string>

// Computer does know well about the User,
// both classes have knowledge of each other
class Computer
{
private:
    std::string m_name;
    User* m_user;

public:
    Computer(std::string name) : m_name{ name }, m_user{ nullptr } {}
    void setUsedBy(User* user) {
        m_user = user;
        // Khi Computer được sử dụng bởi một User,
        // thì cũng có nghĩa `useComputer()` bên trong User cũng được gọi
        user->useComputer(this);
    }
};

// User does know well about the Computer(s),
// both classes have knowledge of each other
class User
{
private:
    std::string m_name;
    std::vector<Computer*> m_computers;

public:
    User(std::string name) : m_name{ name } {}

    void useComputer(Computer* computer) {
      m_computers.push_back(computer);
    }
};

// Driver
int main()  {
    User user{ "Alice" };

    Computer computer{ "Dell" };
    Computer computer1{ "HP" };

    computer1.setUsedBy(&user); // Liên kết User với Computer
    computer2.setUsedBy(&user); // Liên kết User với Computer1

    return 0;
}
```

Liên kết trực tiếp và gián tiếp nhìn chung đều được dùng để biểu diễn quan hệ liên kết giữa các đối tượng lại với nhau, việc chọn lựa liên kết trực tiếp hay gián tiếp còn phải phụ thuộc vào yêu cầu và kiểu thiết kế giữa các đối tượng.

Liên kết trực tiếp nhìn chung thì có vẻ đơn giản hơn, ta có thể dễ hình dung và hiểu hơn về quan hệ giữa các đối tượng có liên quan. Tuy nhiên, nó cũng có những nhược điểm như tăng sự kết dính (tight coupling) giữa các đối tượng, làm cho việc thay đổi hay chỉnh sửa code của một đối tượng có thể làm ảnh hưởng tới những đối tượng có liên kết với nó.

Mặt khác, việc sử dụng liên kết gián tiếp giúp code giữa các đối tượng có liên quan sẽ ít kết dính hơn, bởi vì quan hệ giữa các đối tượng được kết nối trung gian bởi một đối tượng khác. Điều này giúp ta có thể thêm xóa, sửa code mà không sợ bị ảnh hương tới những đối tượng liên quan. Tuy nhiên, liên kết gián tiếp cũng có thể làm cho code trở nên phức tạp hơn, khó hình dung quan hệ giữa các đối tượng.

### Summary

Để có thể dễ nhớ và hình dung những nguyên tắc của các kiểu quan hệ liên kê bên trên, ta có thể tham khảo bảng sau:

| Thuộc tính         | Composition      | Aggregation      | Association        |
| ------------------ | ---------------- | ---------------- | ------------------ |
| Phạm vi quan hệ    | Toàn bộ/một phần | Toàn bộ/một phần | Không liên quan    |
| Quan hệ đồng thời  | Không            | Có               | Có                 |
| Vòng đời ràng buộc | Có               | Không            | Không              |
| Hướng quan hệ      | Đơn hướng        | Đơn hướng        | Đơn hướng/Đa hướng |
| Động từ quan hệ    | Has-a            | Has-a            | Uses-a             |

**_Mình viết blog để tổng hợp lại những gì mình đã học, và cũng như học cách trình bày sao cho người khác có thể hiểu được, nên bài viết có thể tồn tại bug hoặc chưa hoàn thiện đâu đó. Nếu có gì liên quan đến bài viết, cần giúp debug, etc. thì đừng ngần ngại mà hãy cứ nhắn tin cho mình qua [Facebook](https://www.facebook.com/frankiie12a9/) nha._**
