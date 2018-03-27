# Tìm hiểu về Dependecy 

*Dependency injection* và *dependency injection containers* là 2 thứ khác biệt nhau:

- **Dependency injection là một phương thức** viết code tốt hơn.
- **Một container lại là một công cụ** để giúp ta inject các dependency.

Bạn không *cần* một container để thực hiện dependency injection. Tuy nhiên một container sẽ giúp bạn.

PHP-DI viết về: tạo một dependency injection thuần thục hơn.


## Cơ sở lý thuyết

### Code PHP cổ điển

Đây là code **không** sử dụng DI sẽ làm việc như sau:

* App cần Foo (e.g. một Controller), vì vậy:
* App tạo Foo
* App gọi Foo
    * Foo cần Bar (e.g một service), vì vậy:
    * Foo tạo ra Bar
    * Foo gọi Bar
        * Bar cần Bim(1 service, một repository...) vì vậy:
        * Bar tạo Bim
        * Bar thực hiện một số công việc.
        
### Sử dụng denpendcty injection

Đây là cách mà code sử dụng DI sẽ làm việc:

* App cần Foo, cái mà cần Bar(Cái lại cần tới Bim), vì vậy:
* App tạo ra Bim
* App tạo ra Bar và gửi Bim vào đó
* App tạo ra Foo và gửi Bar vào đó
* App gọi Foo
    * Foo gọi Bar
        * Bar làm điều gì đó
        
Đây là khuôn mẫu của **Inversion of Control**. Việc điều khiển của các Dependency được **đảo ngược** từ một dependency đang được gọi tới.

Lợi ích chính: một trong những người đứng đầu chuỗi người gọi luôn là **bạn**. Bạn có toàn quyền quản lí các Dependency và quyết định ứng dụng sẽ chạy ra sao.
Bạn cũng có thể thay thế một dependency bởi một cái khác (ví dụ như một cái bạn tự tạo)

Ví dụ điều gì sẽ xảy ra nếu Library X sử dụng Logger Y và bạn muốn sử dụng Logger Z của bạn? Với Dependency Injection, Bạn không phải thay đổi code của Library X.

### Sử dụng một container

Bây giờ ta sẽ tìm hiểu code sử dụng PHP-DI sẽ làm việc ra sao:

* App cần Foo vì vậy:
* App lấy Foo từ trong Container, do đó:
    * Container tạo ra Bim
    * Container tạo ra Bar và đưa vào đó Bim
    * Container tạo Foo và đưa vào đó Bar
* App gọi Foo
    * Foo gọi Bar
        * Bar làm điều gì đó

Kết luận, **Container sẽ làm hết công việc tạo hoặc inject các dependency**.

## Tìm hiểu thêm với một ví dụ

Đây là một ví dụ thực tế so sánh việc sử dụng (new vs singleton) vs việc sử dụng Dependency Injection.

### Không sử dụng dependency injection

Bạn sẽ có:

```php
class GoogleMaps
{
    public function getCoordinatesFromAddress($address) {
        // calls Google Maps webservice
    }
}
class OpenStreetMap
{
    public function getCoordinatesFromAddress($address) {
        // calls OpenStreetMap webservice
    }
}
```
Theo cách truyền thống sẽ là như sau:

```php
class StoreService
{
    public function getStoreCoordinates($store) {
        $geolocationService = new GoogleMaps();
        // or $geolocationService = GoogleMaps::getInstance() if you use singletons

        return $geolocationService->getCoordinatesFromAddress($store->getAddress());
    }
}
```

Bây giờ chúng ta muốn sử dụng `OpenStreetMap` thay vì `GoogleMaps`, ta sẽ làm ntn?
Ta phải thay đổi code của  `StoreService`, và tất cả những code khác sử dụng `GoogleMaps`.

**Khi không sử dụng Dependency Injection, các class của bạn bị phụ thuộc chặt chẽ vào các dependency của bạn**
                
## Sử dụng Dependency Injection
  
Class `StorService` bây giờ sử dụng Dependency Injection:

```php
class StoreService {
    private $geolocationService;

    public function __construct(GeolocationService $geolocationService) {
        $this->geolocationService = $geolocationService;
    }

    public function getStoreCoordinates($store) {
        return $this->geolocationService->getCoordinatesFromAddress($store->getAddress());
    }
}
```
Và các service đã được xác định sử dụng một interface.  

```php
interface GeolocationService {
    public function getCoordinatesFromAddress($address);
}

class GoogleMaps implements GeolocationService { ...

class OpenStreetMap implements GeolocationService { ...
```
Bây giờ, interface được dành cho người dùng sử dụng StoreService để quyết định xem nên implementation cái nào. Và điều đó có thể thay đổi bất cứ lúc nào,
mà không cần phải viết lại `StoreService`.

**Class `StoreService` không còn bị phụ thuộc quá chặt chẽ vào các dependency của nó nữa.**

## Với PHP-DI

Bạn có thể thấy rằng DI sẽ có một số nhược điểm sau: bạn sẽ phải xử lí inject các dependency

Và Container hay cụ thể là PHP-DI có thể giúp bạn.

Thay vì viết: 

```php
$geolocationService = new GoogleMaps();
$storeService = new StoreService($geolocationService);
```
Bạn có thể viết:

```php
$storeService = $container->get('StoreService');
```

and configure which GeolocationService PHP-DI should automatically inject in StoreService through configuration:

```php
$container->set('GeolocationService', \DI\create('GoogleMaps'));
```

Nếu bạn thay đổi ý định, bây giờ chỉ có một dòng cấu hình để thay đổi.

Thú vị phải không? Hãy tiếp tục và đọc hướng dẫn [Getting Started] (getting-started.md)!
