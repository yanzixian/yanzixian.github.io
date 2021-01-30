---
title: build-my-own-x_simple-database
date: 2021-01-16 13:08:44
categories: c++
tags: build-my-own-x
---

## 一个简单的数据库的实现

跟着[教程](https://cstack.github.io/db_tutorial/)是实现一个最简单的类似SQlite的关系型数据库，该教程使用c语言实现，本文更改为使用c++实现

SQlite的结构如下图所示：

![](https://gitee.com/yanzixian/picBed/raw/master/img202101/20210116162939.png)





---

### Part1 、实现一个REPL

REPL指read-execute-print-loop，是用户与数据库进行交互的程序，是读取命令，执行命令，输出结果的一个循环程序

对于输入流，建立一个结构体`InputBuffer`，使用`read_input`函数从流中读取命令

暂时只定义一个命令`.exit`，输入该命令时结束程序；输入其他字符串时输出错误信息；

程序如下：

```c++
#include <iostream>
#include <cstring>

using namespace std;

const unsigned MAX_INPUT_LENGTH = 50;
typedef struct {
    char buffer[MAX_INPUT_LENGTH];
    int buffer_length;
    int input_length;
} InputBuffer;

InputBuffer *new_input_buffer();

void close_input_buffer(InputBuffer *);

void read_input(InputBuffer *);

void print_prompt();

int main(int argc, char *argv[]) {
    InputBuffer *input_buffer = new_input_buffer();
    while (true) {
        print_prompt();
        read_input(input_buffer);

        if (strcmp(input_buffer->buffer, ".exit") == 0) {
            close_input_buffer(input_buffer);
            return 0;
        } else {
            cout << "Unrecognized command " << input_buffer->buffer << endl;
        }
    }
}

InputBuffer *new_input_buffer() {
    InputBuffer *input_buffer = new InputBuffer();
    input_buffer->buffer_length = 0;
    input_buffer->input_length = 0;

    return input_buffer;
}

void print_prompt() { cout << "db > "; }

void read_input(InputBuffer *input_buffer) {

    cin.getline(input_buffer->buffer, MAX_INPUT_LENGTH);
    int bytes_read = cin.gcount();
    if (bytes_read <= 0) {
        cout << "Error reading input\n";
        return;
    }

    // Ignore trailing newline
    input_buffer->buffer_length = bytes_read;
    input_buffer->input_length = bytes_read - 1;
    input_buffer->buffer[bytes_read - 1] = 0;
}

void close_input_buffer(InputBuffer *input_buffer) {
    delete input_buffer;
}
```

![](https://gitee.com/yanzixian/picBed/raw/master/img202101/20210116222915.png)



---

### Part2 、实现一个简单的SQL编译器和虚拟机

数据库前端交互的输入类型分为两种：

- 元命令，以`.`开头，暂时只有`.exit`

- sql 语句（`sql statement`），类似于`insert`，`select`等数据库操作语句

SQL编译器的作用是把输入的字符串解析为称之`bytecode`的内部表示，`bytecode`传入虚拟机`virtual machine`进行执行

首先，定义两个新的枚举类型，分别表示`meta_command`和`sql statement`的解析结果：

```c++
//meta command解析结果
typedef enum
{
    META_COMMAND_SUCCESS,
    META_COMMAND_UNRECOGNIZED_COMMAND
} MetaCommandResult;

//sql statement 解析结果
typedef enum
{
    PREPARE_SUCCESS,
    PREPARE_UNRECOGNIZED_STATEMENT
} PrepareResult;
```

对于`sql statement`，目前只定义两种类型：

```c++
//statement 类型，暂时只有两种，insert 插入和select 筛选
typedef enum
{
    STATEMENT_INSERT,
    STATEMENT_SELECT
} StateType;

//sql statement 结构定义
typedef struct
{
    StateType type;
} Statement;
```

接下来分别定义对于`meta_command`和`sql statement`的解析函数，返回响应的解析结果：

~~~c++
MetaCommandResult do_meta_command(InputBuffer *input_buffer)
{
    if (strcmp(input_buffer->buffer, ".exit") == 0)
    {
        close_input_buffer(input_buffer);
        cout<<"Exited.\n";
        exit(EXIT_SUCCESS);
    }
    else
    {
        return META_COMMAND_UNRECOGNIZED_COMMAND;
    }
}

PrepareResult prepare_statement(InputBuffer *input_buffer, Statement *statement)
{
    if (strncmp(input_buffer->buffer, "insert", 6) == 0)
    {
        statement->type = STATEMENT_INSERT;

        return PREPARE_SUCCESS;
    }

    if (strncmp(input_buffer->buffer, "select", 6) == 0)
    {
        statement->type = STATEMENT_SELECT;
        return PREPARE_SUCCESS;
    }

    return PREPARE_UNRECOGNIZED_STATEMENT;
}
~~~

对于`sql statement`的解析结果，交由`virtual machine`的执行函数`execute_statement`进行执行：

```c++
void execute_statement(Statement *statement)
{
    switch (statement->type)
    {
        case (STATEMENT_INSERT):
            cout<<"This is where we would do an insert.\n";
            break;
        case (STATEMENT_SELECT):
            cout<<"This is where we would do a select.\n";
            break;
    }
}

```

暂时执行函数只是打印出相关信息

至此，一个简单的SQL编译器和虚拟机完成

完整程序

```c++
//
// Created by yanzx on 2021/1/17.
//

#include <iostream>
#include <cstring>
#include <cstdlib>

using namespace std;

//最大输入长度
const unsigned MAX_INPUT_LENGTH = 50;

//输入流结构定义
typedef struct {
    char buffer[MAX_INPUT_LENGTH];
    int buffer_length;
    int input_length;
} InputBuffer;

//meta command解析结果
typedef enum
{
    META_COMMAND_SUCCESS,
    META_COMMAND_UNRECOGNIZED_COMMAND
} MetaCommandResult;

//sql statement 解析结果
typedef enum
{
    PREPARE_SUCCESS,
    PREPARE_UNRECOGNIZED_STATEMENT
} PrepareResult;

//statement 类型，暂时只有两种，insert 插入和select 筛选
typedef enum
{
    STATEMENT_INSERT,
    STATEMENT_SELECT
} StateType;

//sql statement 结构定义
typedef struct
{
    StateType type;
} Statement;


InputBuffer *new_input_buffer();

void close_input_buffer(InputBuffer *);

void read_input(InputBuffer *);

void print_prompt();

//meta_command解析函数
MetaCommandResult do_meta_command(InputBuffer *input_buffer);
//sql statement 解析函数
PrepareResult prepare_statement(InputBuffer *input_buffer, Statement *statement);
//sql statement 执行函数
void execute_statement(Statement *statement);

int main(int argc, char *argv[]) {
    InputBuffer *input_buffer = new_input_buffer();
    while (true)
    {
        print_prompt();

        read_input(input_buffer);
		//meta_command处理
        if (input_buffer->buffer[0] == '.')
        {
            switch (do_meta_command(input_buffer))
            {
                case (META_COMMAND_SUCCESS):
                    continue;
                case (META_COMMAND_UNRECOGNIZED_COMMAND):
                    cout<<"Unrecognized command"<<input_buffer->buffer<<endl;
                    continue;
            }
        }
		//sql statement处理
        Statement statement;

        switch (prepare_statement(input_buffer, &statement))
        {
            case (PREPARE_SUCCESS):
                break;
            case (PREPARE_UNRECOGNIZED_STATEMENT):
                cout<<"Unrecognized keyword at start of"<<input_buffer->buffer<<endl;
                continue;
        }

        execute_statement(&statement);

        cout<<"Executed.\n";
    }
}

InputBuffer *new_input_buffer() {
    InputBuffer *input_buffer = new InputBuffer();
    input_buffer->buffer_length = 0;
    input_buffer->input_length = 0;

    return input_buffer;
}

void print_prompt() { cout << "db > "; }

void read_input(InputBuffer *input_buffer) {
    int index = 0;
    int bytes_read = 0;
    char ch;
    do {
        cin.get(ch);
        bytes_read++;
        if (ch == ' ') continue;
        input_buffer->buffer[index++] = ch;

    } while (ch != '\n');

    if (bytes_read <= 0) {
        cout << "Error reading input\n";
        exit(EXIT_FAILURE);
    }

    // Ignore trailing newline
    if (index >= MAX_INPUT_LENGTH) {
        cout << "Error: statement too long\n";
        exit(EXIT_FAILURE);
    }
    input_buffer->buffer_length = bytes_read;
    input_buffer->input_length = index;
    input_buffer->buffer[index-1] = 0;
}


void close_input_buffer(InputBuffer *input_buffer) {
    delete input_buffer;
}

MetaCommandResult do_meta_command(InputBuffer *input_buffer)
{
    if (strcmp(input_buffer->buffer, ".exit") == 0)
    {
        close_input_buffer(input_buffer);
        cout<<"Exited.\n";
        exit(EXIT_SUCCESS);
    }
    else
    {
        return META_COMMAND_UNRECOGNIZED_COMMAND;
    }
}

PrepareResult prepare_statement(InputBuffer *input_buffer, Statement *statement)
{
    if (strncmp(input_buffer->buffer, "insert", 6) == 0)
    {
        statement->type = STATEMENT_INSERT;

        return PREPARE_SUCCESS;
    }

    if (strncmp(input_buffer->buffer, "select", 6) == 0)
    {
        statement->type = STATEMENT_SELECT;
        return PREPARE_SUCCESS;
    }

    return PREPARE_UNRECOGNIZED_STATEMENT;
}

void execute_statement(Statement *statement)
{
    switch (statement->type)
    {
        case (STATEMENT_INSERT):
            cout<<"This is where we would do an insert.\n";
            break;
        case (STATEMENT_SELECT):
            cout<<"This is where we would do a select.\n";
            break;
    }
}

```



---

Part3、