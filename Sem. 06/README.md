# **Голямата четворка (Rule of four)**

## [Голямата четворка](https://en.cppreference.com/w/cpp/language/rule_of_three)
 - Конструктор по подразбиране (конструктор без параметри)
 - Конструктор за копиране
 - Оператор =
 - Деструктор
 
Да разгледаме следната структура:
```c++
struct Test {
	char str[20];
	A obj2;
	B obj3;
};
 ```
Понеже функциите (от голямата четворка) не са дефинирани в структурата, то компилаторът ще създаде такива:
```c++
int main() {
	Test currentObject; //default constructor
	 
	Test object2(currentObject); //copy constructor
	 
	currentObject = object2; //operator =

} //destructor (x2)
```
Кодът се компилира успешно и функциите имат правилно поведение.
###  **Как работят дефинираните от компилатора функции?**
Всяка една от тезу функции **извиква рекурсивно същите функции връху член-данните.**

- **Пример за конструктора по подразбиране:**
 
![enter image description here](https://i.ibb.co/s2m8XtC/1.png)
 
- **Пример за деструктора:**

![enter image description here](https://i.ibb.co/kmYSzP7/2.png)

- **Пример за копиращия конструктор:**

![enter image description here](https://i.ibb.co/9Vqk7Mn/3.png)

### **Проблем при функциите, генерирани от компилатора:**

Да разгледаме следния код:

 ```c++
struct Person {
	PersonA(const char* name, int age) {
		this->name = new char[strlen(name) + 1];
		strcpy(this->name, name);
		this->age = age;
	}
private:
	char* name;
	int age;
};

int main() {
	Person p1;
	Person p2(p1);
}
```
Горното извикване на копиращия конструктор има **грешено поведение**:

![enter image description here](https://i.ibb.co/q5rfGBf/Capture.png)


Това е **shallow copy**. В p2 са се копирали стойностите на p1, което довежда до споделяне на една и съща динамична памет.
В тази ситуация ще трябва да се имплементират експлицитно **копиращия конструктор, оператора за присвояване и деструктора**, защото генерираните от компилатора не биха работили правилно.

**Правилното поведение** на копиращия конструктор е следното:

![enter image description here](https://i.ibb.co/XZq5rGT/33.png)

### Собствена имплементация на функциите за копиране и деструктора

 ```c++
struct Person
{
	Person(const char* name, int age) : name(nullptr), age(age) {
		setName(name);
		setAge(age);
	}

	Person(const Person& other) : age(other.age) { // копираме в initializer list нединамичната памет
		copyDynamic(other); // копираме динамична памет
	}

	Person& operator=(const Person& other) {
		if (this != &other) {
			freeDynamic(); //трием динамична памет
			age = other.age; // копираме нединамичната памет
			copyDynamic(other); //копираме динамична памет
		}
		return *this;
	}

	~Person() {
		freeDynamic(); //трием
	}
	
	.
	.
	.
};
```
При всички класове, които използват динамична памет, тези функции изглеждат по този начин. Разликите са в имплементациите на функциите **freeDynamic** и **copyDynamic**.

## Задачи
**Задача 1**: Напишете клас Set, който съдържа множество от числа (без повторения) в диапазона от 0 до n-1, където n е подадено в началото (1 <= n <= 1000). Класът трябва да пази дали съдържа дадено число в битове, т.е ако съдържа дадено число, съответвеният последователен бит ще бъде 1, в противен случай 0. Пример:

{3, 4, 6} => битове на множеството ще бъдат 00011010

{1, 8, 9} => 01000000 11000000

Класът трябва да има следните функции.

- Добавяне на число
- Проверка дали съдържа число
- Принтиране на всички числа, които съдържа
- Принтиране на това как е представено в паметта
- Член-функция, която приема друго множество и връща тяхното обединение
- Член-функция, която приема друго множество и връща тяхното сечение

**Бонус 1**: Направете класът да не зависи от първоначалното n, тоест по-всяко време да можете да добавите, което и да е число >= 0.

**Бонус 2**: Направете функция, която премахва дадено число от множеството.

