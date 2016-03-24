% Неявное приведение типов

Типы могут неявно приводиться к другим типам в определенных контекстах. Эти
изменения в основном - просто *ослабление* типов, сильно сфокусированное на
указателях и временах жизни. Их главная задача - заставить Rust "просто работать"
в большинстве случаев, они в основном безвредны.

Неявное приведение разрешено между следующими типами:

* Транзитивность: `T_1` к `T_3`, где `T_1` неявно приводится к `T_2` и `T_2` 
неявно приводится к `T_3`
* Ослабление Указателей:
    * `&mut T` к `&T`
    * `*mut T` к `*const T`
    * `&T` к `*const T`
    * `&mut T` к `*mut T`
* Неявное приведение к безразмерному типу (ТДР): `T` к `U` если `T` реализует 
`CoerceUnsized<U>`

`CoerceUnsized<Pointer<U>> for Pointer<T> where T: Unsize<U>` реализовано для
всех типов указателей (включая умные указатели, такие как Box и Rc). Неявное 
приведение к безразмерному типу реализуется только автоматически и разрешает 
следующие преобразования:

* `[T; n]` => `[T]`
* `T` => `Trait`, где `T: Trait`
* `Foo<..., T, ...>` => `Foo<..., U, ...>`, где:
    * `T: Unsize<U>`
    * `Foo` - это структура
    * Только у последнего поля `Foo` тип `T`
    * `T` не является частью типа любых других полей

Неявное приведение происходит в *месте неявного приведения*. Любая явно
типизированная область памяти выполняет неявное приведение к ее типу. Неявное
приведение производится не будет, если нужно производить вывод типов. Все места
неявного приведения `e` к типу `U` это:

* Утверждения let, статические переменные и константы: `let x: U = e`
* Аргументы функций: `takes_a_U(e)`
* Любые возвращаемые выражения: `fn foo() -> U { e }`
* Литералы структур: `Foo { some_u: e }`
* Литералы массивов: `let x: [U; 10] = [e, ..]`
* Литералы кортежей: `let x: (U, ..) = (e, ..)`
* Последние выражения в блоке: `let x: U = { ..; e }`

Заметьте, что мы не выполняем неявное приведение при поиске совпадения типажей
(кроме получателей, смотри ниже). Если есть impl для типа `U`, а `T` приводится
к `U`, это не означает, что эта реализация подойдет для `T`. Например, следующее
выражение не пройдет проверку типов, даже при том, что приводить `t` к `&T`
можно и есть impl для `&T`:

```rust,ignore
trait Trait {}

fn foo<X: Trait>(t: X) {}

impl<'a> Trait for &'a i32 {}


fn main() {
    let t: &mut i32 = &mut 0;
    foo(t);
}
```

```text
<anon>:10:5: 10:8 error: the trait `Trait` is not implemented for the type `&mut i32` [E0277]
<anon>:10     foo(t);
              ^~~
```