---
layout: post
title: OOP 101 - Object Relationships (phần 2)
date: 2023-03-30
categories: ["dev", "oop", "cplusplus", "vi"]
---

Ở [phần 1](https://frankiie12a9.github.io/posts/oop-object-relationship-1/#!) chúng ta đã tìm hiểu về những kiểu đối tượng quan hệ như Composition, Aggregation, và Association. Ở phần này, chúng ta sẽ tìm hiểu về những kiểu quan hệ còn lại là Inheritance, và Dependency nhé.

### Object Inheritance

Inheritance là một kiểu quan hệ "is-a", nơi mà một, hoặc nhiều đối tượng kế thừa `thuộc tính`, `methods` từ một hoặc nhiều đối tượng khác. Đối với những đối tượng kế thừa, ta gọi là lớp cha (super class, hay base class), còn đối với những đối tượng được kế thừa, ta gọi là lớp con (subclass hay derived class).

#### Code ví dụ

Ở ví dụ sau đây, chúng ta sẽ dùng chiếc máy tính (Computer), máy tính xách tay (Laptop), và máy tính bàn (Desktop). Máy tính xách tay, hay máy tính bàn nhìn chung đều được coi là một máy tính. Chúng cũng có cho mình những bộ phận chung của một chiếc máy tính như CPU, ổ lưu trữ, Display, hay hệ điều hành... Từ đó, ta có thể thấy máy tính xách tay, và máy tính bàn đều có thể kế thừa những thuộc tính nêu trên từ một đối tượng máy tính.

```c++
#include <iostream>
#include <string>

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
  )
    : m_name{ name }, m_cpu{ cpu }, m_storage{ storage }, m_display{ display }
  {}

  void open() {
    // Open the laptop
  }

  void close() {
    // Close the laptop
  }

  void reboot() {
    // Reboot the laptop
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

class Laptop : public Computer
{
public:
  Laptop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : Computer{name, cpu, storage, display} {}
};

class Desktop : public Computer
{
public:
  Desktop(
    const std::string name,
    const CPU &cpu,
    const Storage &storage,
    const Display &display
  ) : Computer{name, cpu, storage, display} {}
};

int main()
{
  Laptop laptop{
     "Dell DX-3",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1220 * 839, 30} // Display
  };

  Desktop dt{
      "Dell DZ-10",
      {8, 4},           // CPU
      {16, "SSD"},      // Storage
      {1920 * 1080, 50} // Display
  };

  // Either Laptop or Desktop typically is a Computer, and vice versa
  Computer *c1 = &laptop;
  Computer *c2 = &dt;

  std::cout << *c1 << std::endl;
  std::cout << *c2 << std::endl;

  return 0;
}
```

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
    // Run the hardware
  }
};

class OperatingSystem
{
private:
  std::string m_name;

public:
  OperatingSystem(const std::string name) : m_name{ name } {}

  void run() {
    // Run the Operating System
  }
};

class Computer
{
  private:
    Hardware *m_hw;
    OperatingSystem *m_os;

  public:
    // Inject dependencies of `m_hw`, and `m_os` via constructor
    // A.k.a: Constructor Injection
    Computer(hardware *hw, OperatingSystem *os) : m_hw {hw}, m_os {os} {}

    void run() {
      m_hw->run(); // Invoke run() via Dependency of m_hw
      m_os->run(); // Invoke run() via Dependency of m_os

      // Run the Computer
    }
};

// Driver
int main()
{
  Hardware hw{ "Apple" };
  OperatingSystem os{ "Unix" };

  Computer c{"Macbook Pro", &hw, &os};
  c.run();

  return 0;
}
```

### Kết

Kiểu đối tượng quan hệ (Object Relationship) được dùng rất nhiều trong những ngôn ngữ lập trình hướng đối tượng như C++, Java, hay C# nói riêng và lập trình hướng đối tượng nói chung. Việc nắm được những kiến thức cơ bản này sẽ giúp chúng ta rất nhiều trong việc học những [nguyên lí](), hay [design pattern]() phức tạp hơn của lập trình hướng đối tượng.

**_Mình viết blog để tổng hợp lại những gì mình đã học, và cũng như học cách trình bày sao cho người khác có thể hiểu được, nên bài viết có thể tồn tại bug hoặc chưa hoàn thiện đâu đó. Nếu có gì liên quan đến bài viết, cần giúp debug, etc. thì đừng ngần ngại mà hãy cứ nhắn tin cho mình qua [Facebook](https://www.facebook.com/frankiie12a9/) nha._**
