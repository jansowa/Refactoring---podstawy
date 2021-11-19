### 1. "Magic numbers", wpisywanie tej samej wartości w kilku miejscach
<details>
<summary>Przykład "magic numbers"</summary>
<p>

Wpisanie w kodzie bezpośrednio liczby z określonym znaczeniem biznesowym może utrudnić zrozumienie kodu. Warto stosować stałe pomocnicze.
```typescript
if (employee.departmendId === 13 || employee.departmendId === 12) {
    showAdditionalInfos();
}
```
Możemy zmienić na:
```typescript
const hrId = 12;
const infrastructureId = 13;
if (employee.departmentId === hrId || employee.departmentId === infrastructureId) {
    showAdditionalInfos();
}
```

</p>
</details>  
<details>
<summary>Przykład jednej wartości w kilku miejscach</summary>
<p>

Wpisywanie jednej wartości w wielu miejscach utrudnia późniejszą jej zmianę i zrozumienie kodu. Warto wyciągnąć wartość do stałej.
```typescript
export class EmployeeService {
    getWorkersLeaves() {
        return getAllEmployees()
            .filter(employee => employee.type.id === 9)
            .map(employee => employee.leaves);
    }

    getWorkersNames() {
        return getAllEmployees()
            .filter(employee => employee.type.id === 9)
            .map(employee => employee.name);
    }
}
```
Jeśli z dowolnego powodu id typu pracownika "WORKER" zmieni się np. z 9 na 8 - bardzo łatwo się pomylić i zmienić wartość tylko w jednym miejscu. Można od razu zaimplementować klasę w taki sposób:
```typescript
export class EmployeeService {
    readonly workerTypeId = 9

    getWorkersLeaves() {
        return getAllEmployees()
            .filter(employee => employee.type.id === this.workerTypeId)
            .map(employee => employee.leaves);
    }

    getWorkersNames() {
        return getAllEmployees()
            .filter(employee => employee.type.id === this.workerTypeId)
            .map(employee => employee.name);
    }
}
```

</p>
</details>  

##### Narzędzia Visual Studio Code:
- **PPM -> Refactor -> Extract to constant in enclosing scope** - ekstrakcja wartości do stałej

### 2. Dzielenie długich funkcji, metody i zmienne objaśniające

<details>
<summary>Przykład zmiennej objaśniającej skomplikowany kod</summary>
<p>

Rozważny złożony warunek, który sprawdza czy klientowi przysługuje rabat:
```typescript
if ((customer.type.id === newCustomerTypeId && order.summaryPrice > newCustomerMinimalDiscountPrice && isDiscountDate(new Date())) ||
    (customer.type.id === partnerTypeId && order.summaryPrice > partnerMinimalDiscountPrice)) {
    // ... some code
}
```

Trudno w takiej formie zrozumieć, kiedy kod wewnątrz instrukcji warunkowej powinien się wykonać. Warto wyciągnąć wartość logiczną do dobrze nazwanej zmiennej, która ułatwi szybsze zrozumienie:
```typescript
const orderHasDiscount = (customer.type.id === newCustomerTypeId && order.summaryPrice > newCustomerMinimalDiscountPrice && isDiscountDate(new Date())) ||
    (customer.type.id === partnerTypeId && order.summaryPrice > partnerMinimalDiscountPrice);
if (orderHasDiscount) {
    // ... some code
}
```

</p>
</details>

<details>
<summary>Przykład metod objaśniających, dzielących długą funkcję</summary>
<p>

Rozważmy funkcję odpowiedzialną za zamówienie produktu:
```typescript
function orderProduct(itemsWithQuantities: ItemWithQuantity[], customer: Customer) {
    const summaryPrice = itemsWithQuantities
        .map(itemWithQuantity => {
            let itemPrice = getItemPrice(itemWithQuentity.item.id);
            return itemPrice * itemWithQuantity.quantity;
        }).reduce((a, b) => a + b, 0);
    if (summaryPrice < MIN_ORDER_PRICE) {
        return Status.TOO_LOW_PRICE;
    }
    itemsWithQuantities.forEach(
        itemWithQuantity => {
            let itemWarehouseQuantity = getItemWarehouseQuantity(itemWithQuantity.item.id);
            if (itemWarehouseQuantity < itemWithQuantity.quantity) {
                sendAlertToWarehouse(itemWithQuantity.item, itemWithQuantity.quantity);
                return Status.NOT_ENOUGH_IN_WAREHOUSE;
            }
        }
    )
    itemsWithQuantities.forEach(
        itemWithQuantity => {
            setWarehouseQuantity(item.id, getItemWarehouseQuantity(itemWithQuantity.item.id) - itemWithQuantity.quantity)
        }
    )
    let warehouseMail = getMailOrderTemplate();
    warehouseMail.setAddress(ADDRESS.WAREHOUSE);
    warehouseMail.setItems(itemsWithQuantities);
    warehouseMail.setDate(new Date())
    sendMail(warehouseMail);
    let customerMail = getMailOrderTemplate();
    customerMail.setAddress(customer.mail);
    customerMail.setItems(itemsWithQuantities);
    cusomerMail.setDate(new Date());
    sendMail(customerMail);
    
}
```

Na pierwszy rzut oka naprawdę trudno określić, za co odpowiedzialny jest powyższy funkcja. Metoda zawiera w sobie kod wykonujący działania na różnym poziomie abstrakcji. Np. sprawdzając co wykonuje zamówienie - w pierwszym momencie zazwyczaj nie będą nas interesowały szczegóły dotyczące tworzenia wiadomości e-mail.
Warto podzielić kod na mniejsze funkcje:
```typescript
function orderProduct(itemsWithQuantities: ItemWithQuantity[], customer: Customer) {
    checkMinimalPrice(itemsWithQuantities);
    checkItemsAvailability(itemsWithQuantities);
    setWarehouseItemsQuantities(itemsWithQuantities);
    sendOrderMail(itemsWithQuantities, ADDRESS.WAREHOUSE);
    sendOrderMail(itemsWithQuantities, customer.mail);
}
```

</p>
</details>  

##### Narzędzia Visual Studio Code:
- **PPM -> Refactor -> Extract to function in module scope** - ekstrakcja kodu do metody
- **PPM -> Refactor -> Extract to constant in enclosing scope** - ekstrakcja wartości do stałej

### 3. Niezrozumiałe nazwy
##### Narzędzia Visual Studio Code:
- **PPM -> rename symbol** - zmiana nazwy elementu

### 4. DRY (don't repeat yourself) - kopiowanie kodu
Kod nie powinien być powtarzany w kilku miejscach. Jeśli widzimy w kilku miejscach ten sam (czasami też — bardzo podobny) kod w przynajmniej dwóch miejscach — warto zastanowić się nad ekstrakcją metody/zmiennej.
##### Narzędzia Visual Studio Code:
- **PPM -> Refactor -> Extract to constant in enclosing scope** - ekstrakcja wartości do stałej
- **PPM -> Refactor -> Extract to function in module scope** - ekstrakcja kodu do metody

### 5. Nadmiarowe stosowanie if-else, brak 'ternary operator'

<details>
<summary>"return true" / "return false" w konstrukcji if-else</summary>
<p>

Zamiast zwracać true/false wewnątrz instrukcji warunkowych, możemy zwrócić sam warunek (lub jego negację). Przykład:
```typescript
if (condition) {
    return false;
} else {
    return true;
}
```
Zmieniamy na:
```typescript
return !condition
```

</p>
</details>  

<details>
<summary>Przypisanie tej samej wartości w instrukcji warunkowej if-else</summary>
<p>

Możemy zastosować bezpośrednie przypisanie za pomocą 'ternary operator'. Przykład:
```typescript
let c: number;
if (condition) {
  c = 5;
} else {
  c = 3;
}
```
Zmieniamy na:
```typescript
let c = condition ? 1 : 2;
```

</p>
</details>

<details>
<summary>Zwracanie wartości w instrukcji warunkowej if-else</summary>
<p>

Możemy zastosować bezpośrednie zwrócenie wartości za pomocą 'ternary operator'. Przykład:
```typescript
if (condition) {
  return a;
} else {
  return b;
}
```
Zmieniamy na:
```typescript
return  condition ? a : b;
```

</p>
</details>  

#### 6. Martwy kod, zakomentowany kod, nieużywane importy
Wszystkie te elementy są wskazywane przez IDE. Warto traktować wszystkie z nich jako kod do usunięcia - poza sytuacją, w której komentarz wskazuje coś innego.
<details>
<summary>Przykład opisanego kodu w komentarzu</summary>
<p>

```typescript
// Do not remove this code - it will be needed near Christmas
// function showChristmasBanner() {
//     //code
// }
```

</p>
</details>  