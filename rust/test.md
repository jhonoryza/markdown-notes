# Rust Unit Tests

## Unit Tests

- test for only unit of code
- usually place in same place with implementation code

lets create a new lib

```bash
cargo new --lib bank
```

then open with code editor and edit `lib.rs` file like this

```rs
pub struct SavingAccount {
    balance: i32,
}

impl SavingAccount {
    pub fn new() -> SavingAccount {
        SavingAccount { balance: 0 }
    }

    pub fn get_balance(&self) -> i32 {
        self.balance
    }

    pub fn deposit(&mut self, amount: i32) {
        if amount < 0 {
            panic!("Amount should be positive");
        }
        self.balance += amount;
    }

    pub fn transfer(&mut self, amount: i32, target: String) -> Result<String, String> {
        Ok(format!("{} transferred to {}", amount, target))
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn should_have_starting_balance_0() {
        let account = SavingAccount::new();
        assert_eq!(account.get_balance(), 0);
    }

    #[test]
    fn should_be_able_to_deposit() {
        let mut account = SavingAccount::new();
        account.deposit(100);
        assert_eq!(account.get_balance(), 100);
    }

    #[test]
    #[should_panic]
    fn should_not_allow_negative_deposit() {
        let mut account = SavingAccount::new();
        account.deposit(-100);
    }

    #[test]
    fn should_transfer_to_another_account() -> Result<(), String> {
        let mut account = SavingAccount::new();
        account.deposit(100);
        account.transfer(50, "123".to_string())?;
        Ok(())
    }
}
```

to run test run

```bash
cargo test
```

## Integration Tests

- integration test are only test public interface
- test between multiple units of code
- place in root directory called tests

lets create a directory `tests` in root directory then create `saving.rs` file

```rs
mod utils;

use bank::SavingAccount;

#[test]
fn should_have_starting_balance_0() {
    utils::common_setup();
    let account = SavingAccount::new();
    assert_eq!(account.get_balance(), 0);
}
```

create `tests/utils/mod.rs` for common utils file

```rs
pub fn common_setup() {
    println!("Common setup for tests");
}
```

## Documentation

- doc bloc written with `///`
- support markdown
- usefull when you create a library

example

````rs
/// A simple saving account
pub struct SavingAccount {
    balance: i32,
}

impl SavingAccount {
    /// Create a new saving account with a balance of 0
    ///
    /// # Examples
    /// ```
    /// use bank::SavingAccount;
    /// let account = SavingAccount::new();
    /// assert_eq!(account.get_balance(), 0);
    /// ```
    pub fn new() -> SavingAccount {
        SavingAccount { balance: 0 }
    }
}
````

to test doc bloc run

```bash
cargo test
```

to generate the doc

```bash
cargo doc
```

to open generated doc

```bash
cargo doc --open
```
