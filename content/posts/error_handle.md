+++
title = 'rust错误处理'
date = 2023-11-06T06:25:52Z
draft = false
+++

## 使用Anyhow

[anyhow路径](https://docs.rs/anyhow/latest/anyhow/)
anyhow:
库主要是提供一个动态方式的错误包装库, 类似于使用"Result<(), Box<dyn Error>>"的包装

```rust
#[cfg(feature = "std")]
#[cfg_attr(doc_cfg, doc(cfg(feature = "std")))]
impl<E> From<E> for Error
where
    E: StdError + Send + Sync + 'static,
{
    #[cold]
    fn from(error: E) -> Self {
        let backtrace = backtrace_if_absent!(&error);
        Error::from_std(error, backtrace)
    }
}
```

可以从源码看到，anyhow::Error 可以从标准库的Error转换过来，所以可以接受所有实现了标准库Error的错误

## 使用ThisError

[thiserror路径](https://docs.rs/thiserror/latest/thiserror/)
thiserror:
库主要基于过程宏，方便的增加对Error trait的快速实现
``` rust
use std::{error::Error, fmt::Debug};

fn main() {
    println!("{:?}", make_request().unwrap_err());
}

fn make_request() -> Result<(), CustomError> {
    use CustomError::*;

    let key = std::fs::read_to_string("some-key-file").map_err(FileReadError)?;
    reqwest::blocking::get(format!("http:key/{}", key))?.error_for_status()?;
    std::fs::remove_file("some-key-file").map_err(FileDeleteError)?;
    Ok(())
}

#[derive(thiserror::Error)]
enum CustomError {
    #[error("failed to read the key file")]
    FileReadError(#[source] std::io::Error),

    #[error("failed to send the api request")]
    RequestError(#[from] reqwest::Error),

    #[error("failed to delete the key file")]
    FileDeleteError(#[source] std::io::Error),
}

impl Debug for CustomError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        writeln!(f, "{}", self)?;
        if let Some(source) = self.source() {
            writeln!(f, "Caused by:\n\t{}", source)?;
        }
        Ok(())
    }
}

```
上面例子代码中
* \#[error] 定义了Display表示
* \#[from] 定义了自动类型转换的源类型
* \#[source] 源定义了和标准库中的Error中source函数一致的表示，即低一层的错误源,这里例子特别注意由于from是不允许不同类型的枚举值，从同一个错误类型转换过来，所以上面std::io::Error只能定义为source

### 使用建议
anyhow 主要利用trait obj将错误信息存储，所以错误可以是任意实现了Error trait的的错误。缺点是需要动态类型转换，错误类型在编译器不能完全确定。适合在应用层使用，而不适合作为组件库中使用。应用层可以不定义错误类型,直接使用。
ThisError 主要通过Drive宏作为语法糖，简化开发实现，强类型，适合作为库开发中使用。

### 参考：
[How to Use the “thiserror” Crate in Rust](https://betterprogramming.pub/a-simple-guide-to-using-thiserror-crate-in-rust-eee6e442409b)
