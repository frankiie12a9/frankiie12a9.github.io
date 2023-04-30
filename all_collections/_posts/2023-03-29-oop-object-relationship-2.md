---
layout: post
title: OOP 101 - Object Relationships (phần 2)
date: 2023-03-30
categories: ["dev", "oop", "cplusplus", "vi"]
---

Ở [phần 1](https://frankiie12a9.github.io/posts/oop-object-relationship-1/#!) chúng ta đã tìm hiểu về những kiểu đối tượng quan hệ như Composition, Aggregation, và Association. Ở phần này, chúng ta sẽ tìm hiểu về những kiểu quan hệ còn lại là Inheritance, và Dependency nhé.

### Object Inheritance

Inheritance là một kiểu quan hệ "is-a", nơi mà một, hoặc nhiều đối tượng kế thừa `thuộc tính`, `methods` từ một hoặc nhiều đối tượng khác. Đối với những đối tượng kế thừa, ta gọi là lớp cha/lớp cơ sở (super class, hay base class), còn đối với những đối tượng được kế thừa, ta gọi là lớp con/lớp dẫn xuất (subclass hay derived class).

#### Code ví dụ

Ở ví dụ sau đây, chúng ta sẽ dùng chiếc `máy tính (Computer)`, `máy tính xách tay (Laptop)`, và `máy tính bàn (Desktop)`. Máy tính xách tay, hay máy tính bàn nhìn chung đều được coi là một máy tính. Chúng cũng có cho mình những bộ phận chung của một chiếc máy tính như CPU, ổ lưu trữ, Display, hay hệ điều hành... Từ đó, ta có thể thấy máy tính xách tay, và máy tính bàn đều có thể kế thừa những thuộc tính nêu trên từ một đối tượng máy tính.

```c++
#include <iostream>
#include <string>

// Computer là lớp cha
class Computer
{
protected:
  std::string m_name;
  CPU m_cpu;
  Storage m_storage;
  Display m_display;

public:
  Computer(
    const std::string &name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : m_name{ name }, m_cpu{ cpu }, m_storage{ storage }, m_display{ display } {}

  void open() {
    std::cout << "Open the laptop" << std::endl;
  }

  void close() {
    std::cout << "Close the laptop" << std::endl;
  }

  void reboot() {
    std::cout << "Reboot the laptop" << std::endl;
  }

  // Overloaded output method
  friend std::ostream& operator<<(std::ostream& out, const Computer& computer)
  {
    out << computer.m_name << std::endl;
    out << "CPU (" << "clock_speed: " << computer.m_cpu.clock_speed << ", core_cout: " << computer.m_cpu.core_count << ")" <<  std::endl;
    out << "Storage (" << "size: " << computer.m_storage.size << ", type: " << computer.m_storage.type << ")" << std::endl;
    out << "Display (" << "resolution: " << computer.m_display.resolution << ", refresh_rate: " << computer.m_display.refresh_rate << ")" << std::endl;
    return out;
  }
};

// Laptop kế thừa từ Computer, nên Laptop là lớp con
class Laptop : public Computer
{
public:
  Laptop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  )
  // kế thừa thuộc tính từ Computer
  : Computer{name, cpu, storage, display} {}
};

// Desktop kế thừa từ Computer, nên Desktop là lớp con
class Desktop : public Computer
{
public:
  Desktop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  )
  // kế thừa thuộc tính từ Computer
  : Computer{name, cpu, storage, display} {}
};

int main()
{
  Laptop lt{
     "Dell DX-3",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1220 * 839, 30}  // Display
  };

  Desktop dt{
      "Dell DZ-10",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1920 * 1080, 50} // Display
  };

  // Sau khi được kế thừa từ Computer.
  // Lúc này, đối tượng Computer chính là Laptop, và cũng chính là Desktop
  Computer *c1 = &lt;
  Computer *c2 = &dt;

  std::cout << *c1 << std::endl;
  std::cout << *c2 << std::endl;

  return 0;
}
```

<!-- <details>
  <summary>LƯU Ý THÊM</summary>
  <strong>strongly important</strong> and this text is <em>emphasized</em>
</details> -->

> **Ở ví dụ dược trình bày trên, liệu bạn có tự hỏi rằng, ở hàm tạo tham số trong hai lớp `Laptop`, và `Desktop`, tại sao chúng ta lại khai báo kế thừa thuộc tính thành viên từ lớp Computer thông qua hàm tạo tham số `: Computer{ name, cpu, storage, display } {}`? Nó có vai trò gì? Sẽ ra sao nếu chúng ta lược bỏ nó?**

Ở lớp Computer, chúng ta thấy nó có một hàm tạo tham số (parameterized constructor), nơi mà những thuộc tính của một Computer được khai báo và gán giá trị với những tham số tương ứng.

```c++
Computer(
    const std::string &name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
) : m_name{ name }, m_cpu{ cpu }, m_storage{ storage }, m_display{ display } {}
```

Và ở trong hai lớp kế thừa từ Computer là Laptop, và Desktop, chúng cũng có hai hàm tạo tham số như:

```c++
class Laptop : public Computer
{
public:
  // Hàm tạo tham số
  Laptop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : Computer{ name, cpu, storage, display } {}
};

class Desktop : public Computer
{
public:
  // Hàm tạo tham số
  Desktop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : Computer{ name, cpu, storage, display } {}
};
```

Giả sử ta ta xóa đi dòng `: Computer{ name, cpu, storage, display }`, Thì chức năng của hai lớp Laptop, và Desktop sẽ có những ảnh hưởng sau đây:

- <h5>Lỗi biên dịch ở Laptop, và Desktop</h5>

  Với code được trình bày ở bên trên, nếu ta xóa `: Computer{ name, cpu, storage, display }`
  thì ngay lập tức một lỗi biên dịch là `no default constructor exists for class "Computer"`. Lỗi này ám chỉ rằng, khi lớp Laptop, Desktop kế thừa từ Computer, _hàm tạo của lớp cơ sở mà chúng kế thừa (Computer) sẽ được gọi trước tiên_ để khởi tạo các thành viên của nó. Nếu hàm tạo tham số hiện tại của Computer không được gọi, thì hàm tạo mặc định của nó sẽ được gọi. Tuy nhiên trong trường hợp này, ở Computer không có hàm tạo mặc định nào cả, mà chỉ có duy nhất một hàm tạo tham số, thế nên lỗi trên bị nhả ra như một điều tất yếu.

  ```c++
  // Hàm tạo tham số của Laptop
   Laptop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  )
  // : Computer{ name, cpu, storage, display }
  {} // ERROR!

  // Hàm tạo tham số của Desktop
  Desktop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  )
  // : Computer{ name, cpu, storage, display }
  {} // ERROR!


  // Hiện tại lớp Computer không có hàm tạo mặc định
  Computer() = default;

  // Mà chỉ có duy nhất hàm tạo tham số
  Computer(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : m_name{ name }, m_cpu{ cpu }, m_storage{ storage }, m_display{ display } {}
  ```

  Để có thể khắc phục được lỗi này, chúng ta cần thêm một hàm tạo mặc định cho lớp Computer như sau:

  ```c++
  class Computer
  {
    // ...

  public:
    // Thêm hàm tạo mặc định
    Computer() = default;

    // ...
  }
  ```

- <h5>Thành viên kế thừa bị ảnh hưởng</h5>

  Nếu ta không định nghĩa `: Computer{ name, cpu, storage, display }` trong hàm tạo của lớp dẫn xuất (Laptop, Desktop), thì những biến thành viên như `m_name`, `m_cpu`, `m_storage`, và `m_display` của lớp Computer sẽ không được khởi tạo. Điều này có nghĩa Laptop sẽ kế thừa những thành viên `m_name`, `m_cpu`, `m_storage`, và `m_display` có giá trị rỗng, tức chưa được khởi tạo và gán giá trị.

  ```c++
  class Laptop : public Computer
  {
  public:
    Laptop(
      const std::string name,
      const CPU &cpu,
      const Storage &storage,
      const Display &display
    )
    // Giả sử ta xóa dòng này
    // : Computer{ name, cpu, storage, display }
    {}

    // Thì thành viên `m_name` sẽ là rỗng, tức không được khai báo và gán giá trị.
    // `m_cpu`, `m_storage`, hay `m_display` lúc này cũng sẽ là rỗng
    std::string getName() { return this->m_name; }
  };
  ```

Bản chất của quan hệ kế thừa là lớp con (sub/derived class) sẽ kế thừa và "phát huy" những gì mà lớp cha (super/base class) đang sở hữu. Việc nắm rõ được bản chất của mối quan hệ giữa lớp cha và con, cũng như cách mà các thành viên trong chúng được kế thừa giúp ta có thể thiết kế ra những lớp phức tạp hơn, code sẽ ít lặp lại hơn, có thể sử tái sử dụng, cũng như mà mở rộng nếu cần.

### Object Dependencies

Dependency là một kiểu quan hệ, nơi mà một hoặc nhiều đối tượng phụ thuộc đối tượng khác để thực thi nhiệm vụ của nó. Cũng giống với Aggregation, và Associtation, các đối tượng trong quan hệ Dependency, tồn tại độc lập, và vòng đời của đối tượng phụ thuộc không bị ràng buộc bởi đối tuợng bị phụ thuộc. Đối với kiểu quan hệ này, chúng ta có thể dễ dàng thấy nó được dùng để minh họa trong những ví dụ, một chiếc `máy tính (Computer)` có thể sử dụng `hệ điều hành (Operating System)` Windows, hoặc Unix, Linux, việc dùng hệ điều nào đi chăng nữa thì cũng không ảnh hưởng tới chức năng của máy. Hệ điều hành chỉ là một phần được gắn vào máy tính để phục vụ những chức năng mà nó mang lại.

#### Code ví dụ

Ở ví dụ sau đây, chúng ta sẽ minh họa thông qua chiếc `máy tính (Computer)`, `phần cứng (Hardware)`, và `hệ điều hành (Operating System)`. Để máy tính có thể thực hiện được những công việc mang tính chất vật lí, chúng cần đến những bộ phận như CPU, ổ lưu trữ (Storage),.. những bộ phận này thuộc về phần cứng, và để quản lí được phần cứng, cũng như hệ sinh thái phần mềm, máy tính phải cần đến hệ điều hành. Qua những điều trên cho ta thấy, máy tính sẽ phải phụ thuộc vào phần cứng, và hệ điều hành.

```c++
#include <iostream>
#include <string>

class Hardware
{
private:
  std::string m_name;

public:
  Hardware(const std::string name) : m_name{ name } {}

  void run() {
    std::cout << "Run the hardware" << std::endl;
  }
};

class OperatingSystem
{
private:
  std::string m_name;

public:
  OperatingSystem(const std::string name) : m_name{ name } {}

  void run() {
    std::cout << "Run the Operating System" << std::endl;
  }
};

class Computer
{
  private:
    Hardware *m_hw;
    OperatingSystem *m_os;

  public:
    // Gán sự phụ thuộc `m_hw`, `m_os` thông qua hàm tạo.
    // Kĩ thuật này còn được gọi là Constructor Injection
    Computer(hardware *hw, OperatingSystem *os) : m_hw{ hw }, m_os{ os } {}

    void run() {
      m_hw->run(); // Ta có thể gọi hàm `run()` thông qua việc gán phụ thuộc `m_hw`
      m_os->run(); // Ta có thể gọi hàm `run()` thông qua việc gán phụ thuộc `m_os`

      std::cout << "Run the Computer" << std::endl;
    }
};

// Driver
int main()
{
  Hardware hw{ "Apple" };
  OperatingSystem os{ "Unix" };

  Computer c{ "Macbook Pro", &hw, &os };
  c.run();

  return 0;
}
```

### Kết

Kiểu đối tượng quan hệ (Object Relationship) được dùng rất nhiều trong những ngôn ngữ lập trình hướng đối tượng như C++, Java, hay C# nói riêng và lập trình hướng đối tượng nói chung. Việc nắm được những kiến thức cơ bản này sẽ giúp chúng ta rất nhiều trong việc học những [nguyên lí](https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english/), hay [design pattern](https://www.bing.com/ck/a?!&&p=cf932fb5745e49eeJmltdHM9MTY4MjY0MDAwMCZpZ3VpZD0yNTA3YTg3NC1hM2NiLTY3NGQtMjc0OC1iYWU0YTI0ZDY2YTgmaW5zaWQ9NTIyMg&ptn=3&hsh=3&fclid=2507a874-a3cb-674d-2748-bae4a24d66a8&psq=OOP+design+patterns&u=a1aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvRGVzaWduX1BhdHRlcm5z&ntb=1) phức tạp hơn của lập trình hướng đối tượng.

### Đọc thêm

- [OOP 101: Object Relationships (phần 1)](https://frankiie12a9.github.io/posts/oop-object-relationship-1/#!)

**_Mình viết blog để tổng hợp lại những gì mình đã học, và cũng như học cách trình bày sao cho người khác có thể hiểu được, nên bài viết có thể tồn tại bug hoặc chưa hoàn thiện đâu đó. Nếu có gì liên quan đến bài viết, cần giúp debug, etc. thì đừng ngần ngại mà hãy cứ nhắn tin cho mình qua [Facebook](https://www.facebook.com/frankiie12a9/) nha._**
