---
layout: post
title: Cùng tìm hiểu những khái niệm về Web Server, và những Internet Protocols thường dùng.
date: 2023-03-01
categories: ["cloud", "web server", "vi"]
---

<!-- todo -->

### References:

- https://www.digitalocean.com/community/tutorials/web-servers-checkpoint

<!-- todo -->

### 4. Object Inheritance

Inheritance is an "is-a" relationship where one class (the subclass) inherits the properties and behaviors of another class (the superclass). The subclass can add its own properties and behaviors as well. For example, a car class might inherit from a vehicle class, and add its own properties like "number of doors" and "color".

Inheritance là một kiểu quan hệ "is-a", nơi mà một, hoặc nhiều đối tượng kế thừa `thuộc tính`, `methods` từ một hoặc nhiều đối tượng khác. Đối với những đối tượng kế thừa, ta gọi là lớp cha (super class, hay baseclass), còn đối với những đối tượng được kế thừa, ta gọi là lớp con (subclass hay derived class)

```c++
class Computer
{
protected:
  CPU m_processor;
  Storage m_storage;
  Display m_display;

public:
  public Computer(
    const std::string &name,
    const CPU &CPU,
    const Storage &storage,
    const Display &display )
      : m_name{ name }, m_processor{ CPU }, m_storage{ storage }, m_display{ display }
  {}

  void run() {
    // Run the computer
  }
};

class Laptop : public Computer
{
  public:
    void open() {
      // Open the laptop
    }

    void close() {
      // Close the laptop
    }

    void reboot() {
      // Reboot the laptop
    }
};

class Desktop : public Computer
{
  public:
    void open() {
      // Open the Desktop
    }

    void close() {
      // Close the Desktop
    }

    void reboot() {
      // Reboot the Desktop
    }
};
```

### 5. Object Dependency

Dependency is a relationship where one object depends on another object to perform its task, but the lifetime of the depended object is not tied to the lifetime of the depending object. For example, a car "depends-on" a fuel pump to pump fuel, but the fuel pump can be replaced without affecting the car's lifetime.

```c++

class Hardware
{
  public:
    void run() {

    }
}

class OperatingSystem
{
  public:
    void run() {

    }
}

class Computer
{
  private:
    Hardware *m_hw;
    OperatingSystem *m_os;

  public:
    Computer(hardware *hw, OperatingSystem *os) : m_hw {hw}, m_os {os} {}

    void run() {
      // Use the operating system and hardware to run the computer.
    }
}

```

### 6. Container Class
