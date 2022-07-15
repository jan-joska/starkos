# Typy v .NET

.NET definuje čtyři základní typy 
- Hodnotový typ
- Referenční typ
- Record
- Ref Record

## Hodnotový typ

Je datový typ, který uchovává hodnotu ve své vlastí paměťové alokaci. Hotnotový typ je definován hodnotami, které obsahuje a bez nich postrádá smysl.
Hodnotový typ se vytvoří pomocí klíčového slova `struct`
Struktura nemusí být imutabilní, ale nejlepších výsledků se dosahuje, když imutabilní je (problémy s equals, ukládání do hash struktur apod.)
Struktura nesmí mít bezparametrický konstruktor (To by umožnilo vytvořit strukturu bez hodnoty - nemá smysl)
Hodnotový typem jsou v BCL:
- všechny numerické typy
- `bool`, `char`, `date`
- všechný struktury i když jejich členové jsou referenční typy
- enumerace (na pozadí jsou to celočíselné typy `sbyte`, `short`, `int`, `long` apod.)

### Předávání hodnotového typu

Nejzásadnější rozdíl mezi hodnotovým typem a typem referenčním je ve způsobu jejich předávání.
Pakliže je hodnotový typ předáván, vytváří se kopie typu spolu se všemi jeho členy. Původní typ zůstává zachován. Změny provedené na kopii typu se do originálu nepromítnou. Toto chování lze změnit za použití klíčového slova [`ref`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#passing-an-argument-by-reference)

``` csharp
// Ačkoliv je int hodnotový typ, užití klíčového slova ref způsobí předání odkazem.
// Úprava takto předané proměnné se projeví i v původní hodnotě
void Method(ref int refArgument)
{
    refArgument = refArgument + 44;
}
```

### Porovávání hodnotového typu

Hodnotový type (poděděnec z ValueType) přepisuje chování metody `Equals(Object)` zděděné z `Object` tak, že dva hodnotové typy jsou si rovny, pokud jsou si rovny všechny jeho členy.

K porovnávání polí hodnotového typu se využívá reflexe. Výchozí implementace není efektivní a je dobrý nápad hodnotovému typu přepsat metodu `Equals(object)` a `GetHashCode`.

Někdy může být výhodné přepsat chování equals hodnotového typu, aby se dosáhlo užitečnější implementace např. když:
- Chceme kontrolovat porovnávání velkých a malých písmen u obsaženého stringu
- Nevyhovuje nám výchozí způsob porovnávání referenčních typů přítomných v hodnotovém typu jako fields (výchozí porovnání referenčního typu je na rovnost referencí)

``` csharp
public struct Vector
{
    public readonly int X { get; }
    public readonly int Y { get; }

    public Vector (int x, int y)
    {
        X = x;
        Y = y;
    }

    public override bool Equals(object? obj)
    {
        if (obj is Vector other)
        {
            return X.Equals(other.X) && Y.Equals(other.Y);
        }
        return false;
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(X, Y);
    }
}
```

### Kdy vytvořit typ jako strukturu

- Logicky reprezentuje jedinou hodnotu tak jako primitivní typy
- Má instanční velikost (úhrn velikosti všech fields {Při vytváření struktury nevzniká žádná režie, hodnoty jsou v paměti umístěny těsně za sebe bez hlavičky apod.}) pod 16 bajtů
- Je imutabilní
- Nemusí se často boxovat (ukládání hodnotového typu do proměnné typu object)

### Boxování hodnotového typu

Následující kód způsobí alokaci místa na haldě, které so po zániku reference musí podrobit svozu odpadků. Při nadužívání boxování velkého množství hodnotových typů to může vést ke snížení výkonosti.
``` csharp
// Boxed in reference type
object v = new Vector();

// Unboxed 
int v = (Vector)v;
```
### Vestavěné hodnotové typy

.NET core poskytuje velké množství předpřipravených hodnotových typů, které se dají rozdělit do těchto kategorií:

- Celočíselné typy
- Typy pro čísla s plovoucí čárkou
- Bool
- Char
- Enumerátory
- Tuples (ntice)
- Nulovatelné hodnotové typy


https://www.c-sharpcorner.com/article/C-Sharp-heaping-vs-stacking-in-net-part-i/

https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#passing-an-argument-by-reference

https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct

https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/struct

https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types