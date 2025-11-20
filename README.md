# COSModule_2019_Guide
Aadarsha

# Qt/C++ Mini Guide — Questions 1–4

## Question 1 — Input dialog, splitting and message boxes

Code (cleaned):

```cpp
QString userInput = QInputDialog::getText(0, "User Details",
                                          "Enter your full name (each word separated by :)");
QStringList userInfo = userInput.split(":");
if (userInfo.length() < 2)
    QMessageBox::warning(0, "Error", "Enter your full name!", 0, 0);
else
    QMessageBox::information(0, "Result",
                             QString("Your name consists of %1 words.")
                             .arg(userInfo.length()),
                             0, 0);

// Example input: "Aadarsha:Poovalingam:Aadarsha:Poovalingam"
// userInfo becomes {"Aadarsha", "Poovalingam", "Aadarsha", "Poovalingam"}
```

### 1.1 Example of a non-empty input that triggers the error
Any single word without `:` will result in `userInfo.length() == 1 < 2` and display the warning.

Example:
```
Mike
```

This will show:
```
Enter your full name!
```

### 1.2 Example: `Mike:William:Owen`
- `userInfo.length() == 3`
- The information box shown:
```
Your name consists of 3 words.
```

### 1.3 Entering `4444`
- `QInputDialog::getText` treats input as text.
- `userInput.split(":")` yields `{`"4444"`}`.
- `userInfo.length()` is 1 → the warning appears:
```
Enter your full name!
```

### 1.4 One similarity and one difference between QInputDialog and QMessageBox
- Similarity: Both are modal dialog windows used to interact with the user; they block until the user responds.
- Difference:
  - `QInputDialog` provides an editable field for the user to enter typed data (text, number, etc.).
  - `QMessageBox` is for displaying messages and getting simple button responses (OK/Yes/No) and does not provide text entry.

---

## Question 2 — Transactions class hierarchy and TransactionList

### 2.1 Abstract vs concrete classes
- Abstract: `Transaction` (intended to be abstract; contains `computeCost()` to be overridden).
- Concrete: `Deposit`, `Withdrawal`.

### 2.2 Constructor for `Withdrawal`
Example implementation that calls the base constructor and initializes members:

```cpp
Withdrawal::Withdrawal(double amount)
    : Transaction("Withdrawal", QDateTime::currentDateTime()),
      m_Amount(amount),
      m_Percentage(0.0) // or other sensible default
{
}
```

Key points:
- Call the base `Transaction` constructor with `"Withdrawal"` and a timestamp.
- Initialize withdrawal-specific members.

### 2.3 toString implementations

(a) `Transaction::toString()` — include type and date/time:

```cpp
QString Transaction::toString() const
{
    return QString("Type: %1, Date/time: %2")
            .arg(m_Type)
            .arg(m_DateTime.toString());
}
```

(b) `Deposit::toString()` — partial override calling base version:

```cpp
QString Deposit::toString() const
{
    QString s = Transaction::toString();
    s += QString(", Amount: %1, Fee: %2")
             .arg(m_Amount)
             .arg(m_Fee);
    return s;
}
```

Main ideas:
- Reuse base `toString()` and append derived-class details.

### 2.4 Correctness of pointer usage snippets

```cpp
(a) Withdrawal *t1 = new Withdrawal(120.25); // Correct
(b) Deposit    *t2 = new Transaction("Deposit", QDateTime::currentDateTime()); // Incorrect — cannot assign base to derived; Transaction is likely abstract
(c) Transaction *t3 = new Deposit(2000); // Correct — base pointer to derived object (polymorphism)
```

### 2.5 TransactionList — selected methods

Given:

```cpp
class TransactionList{
public:
    ~TransactionList();
    void addTransaction(Transaction* t);
    double totalTransactionCost() const;
    QString frequentTransactionType() const;
    QList<Transaction*> transactionsOnADate(QDate date) const;
    QString toString() const;
private:
    QMap<QDateTime, Transaction*> m_TransactionList;
};
```

(a) addTransaction:

```cpp
void TransactionList::addTransaction(Transaction* t)
{
    m_TransactionList.insert(t->getDateTime(), t);
}
```

(b) totalTransactionCost:

```cpp
double TransactionList::totalTransactionCost() const
{
    double total = 0.0;
    for (auto it = m_TransactionList.constBegin(); it != m_TransactionList.constEnd(); ++it)
        total += it.value()->computeCost();
    return total;
}
```

(c) Destructor showing composition (TransactionList owns the transactions):

```cpp
TransactionList::~TransactionList()
{
    for (auto it = m_TransactionList.begin(); it != m_TransactionList.end(); ++it)
        delete it.value();
    m_TransactionList.clear();
}
```

Deleting contained `Transaction*` objects signals ownership (composition).

---

## Question 3 — Virtual vs pure virtual, constructors, design patterns

### 3.1 Differences in UML and C++
- UML: Abstract/pure-virtual operations sometimes italicised; abstract class also italicised.
- C++:
  - Virtual function: `virtual void f();` — can have an implementation; derived classes may override.
  - Pure virtual: `virtual void f() = 0;` — forces derived classes to implement; makes the class abstract.

### 3.2 Why an abstract class needs a constructor
- Even though abstract classes cannot be instantiated directly, their constructors initialize base-class members and are invoked when constructing derived objects.

### 3.3 Benefits of design patterns
- Provide reusable, proven solutions.
- Improve team communication by shared vocabulary.
- Promote maintainable, extensible designs and reduce bugs.

### 3.4 Observer pattern and Qt
- Observer defines a one-to-many dependency: when subject changes, observers are notified.
- In Qt, signals and slots implement the Observer pattern:
  - A signal is emitted by the subject.
  - Connected slots in observers are called automatically.

### 3.5 Classes in the Composite pattern
- Component
- Leaf
- Composite

Clients treat Leaf and Composite uniformly through the Component interface.

---

## Question 4 — FortuneCookie dialog and related GUI behavior

Definition and implementation (cleaned):

```cpp
class FortuneCookie : public QDialog {
    Q_OBJECT
private:
    QStringList messages;
public:
    FortuneCookie();
public slots:
    void showFortune();
};

// implementation
FortuneCookie::FortuneCookie() {
    messages << "Express yourself. Don't hold back"
             << "For success today look first at yourself"
             << "Whenever possible, keep it simple";

    QPushButton *pb = new QPushButton("Open a fortune cookie");
    QVBoxLayout *layout = new QVBoxLayout;
    layout->addWidget(pb);
    this->setLayout(layout);

    connect(pb, SIGNAL(clicked()), this, SLOT(showFortune()));
}

void FortuneCookie::showFortune() {
    QString m = messages[rand() % 3];
    QMessageBox::information(this, "Message", m);
}
```

### 4.1 What it does
- Shows a dialog with a button "Open a fortune cookie".
- When clicked, selects one of three messages at random and displays it in a QMessageBox.

### 4.2 Purpose of signals and slots here
- The button emits `clicked()`; `connect(pb, SIGNAL(clicked()), this, SLOT(showFortune()));` routes that event to `showFortune()`. Signals and slots provide a type-safe, loosely coupled event system.

### 4.3 Why `messages` is a data member but button/layout are locals
- `messages` must persist because it is used after construction (in `showFortune()`).
- The button and layout are only needed to set up the UI; Qt's parent–child ownership ensures they are deleted with the dialog, so locals are fine.

### 4.4 Why `Q_OBJECT` is required
- Enables Qt's meta-object system necessary for signals/slots and runtime type information. Without it, signal-slot connections and other meta features won't work.

### 4.5 Are heap allocations in the constructor a problem?
- Not a problem: `layout` is set on `this` and `pb` is added to the layout. Qt parent-child ownership ensures they are deleted when the dialog is destroyed.

### 4.6 Parent–child object tree
- `FortuneCookie` (dialog)
  - `QVBoxLayout` (layout)
  - `QPushButton` (pb)

### 4.7 Variant: periodically showing messages with user interval
One correct approach uses QLineEdit for the interval, a Start button and a QTimer:

```cpp
class FortuneCookie : public QDialog {
    Q_OBJECT
private:
    QStringList messages;
    QLineEdit   *intervalEdit;
    QPushButton *startButton;
    QTimer      *timer;
public:
    FortuneCookie(QWidget *parent = 0);

public slots:
    void showFortune();
    void startFortunes();
};

// implementation
FortuneCookie::FortuneCookie(QWidget *parent)
    : QDialog(parent)
{
    messages << "Express yourself. Don't hold back"
             << "For success today look first at yourself"
             << "Whenever possible, keep it simple";

    intervalEdit = new QLineEdit;
    intervalEdit->setPlaceholderText("Interval (ms)");
    startButton = new QPushButton("Start");
    timer = new QTimer(this);

    QVBoxLayout *layout = new QVBoxLayout;
    layout->addWidget(intervalEdit);
    layout->addWidget(startButton);
    setLayout(layout);

    connect(startButton, SIGNAL(clicked()), this, SLOT(startFortunes()));
    connect(timer, SIGNAL(timeout()), this, SLOT(showFortune()));
}

void FortuneCookie::startFortunes()
{
    int interval = intervalEdit->text().toInt(); // milliseconds
    if (interval > 0)
        timer->start(interval);
}

void FortuneCookie::showFortune()
{
    QString m = messages[qrand() % messages.size()]; // or QRandomGenerator
    QMessageBox::information(this, "Message", m);
}
```

Notes:
- Use a proper random generator (e.g., QRandomGenerator or seed qrand()).
- Interval is read from user input and used to start the QTimer.

### 4.8 MyMainWindow constructor (menu + action)
Example constructor that adds an "applications" menu and a "Fortune Cookie" action:

```cpp
MyMainWindow::MyMainWindow()
{
    QMenu *appMenu = menuBar()->addMenu("applications");
    QAction *fortuneAction = appMenu->addAction("Fortune Cookie");
    connect(fortuneAction, SIGNAL(triggered()),
            this, SLOT(showFortuneCookie()));
}
```

Selecting the menu item triggers `showFortuneCookie()`.

---
