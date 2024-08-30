# SSC-Internship-Dev-ASP.NET-Controllers-REST-API
# Hướng Dẫn Về Controllers & REST API Trong ASP.NET Core
## Giới Thiệu
Controllers là trung tâm của một ứng dụng ASP.NET Core MVC và đóng vai trò là cầu nối giữa người dùng và logic kinh doanh. Chúng xử lý các yêu cầu HTTP, tương tác với các dịch vụ, và trả về phản hồi cho người dùng dưới dạng HTML, JSON, hoặc các định dạng khác.

## 1. Controllers Là Gì?
Controllers là các lớp trong ASP.NET Core xử lý các yêu cầu từ client. Chúng chứa các phương thức, được gọi là "actions", để thực thi các tác vụ khác nhau như lấy dữ liệu từ cơ sở dữ liệu, xử lý logic kinh doanh, và trả về kết quả cho client. Controllers đóng vai trò quản lý luồng dữ liệu giữa view (giao diện người dùng) và mô hình dữ liệu.

Mỗi phương thức trong Controller thường ánh xạ đến một hành động cụ thể của người dùng, như yêu cầu xem một trang, đăng dữ liệu qua biểu mẫu, hoặc truy xuất dữ liệu thông qua API.

## 2. Tạo Controllers Trong ASP.NET Core
Bạn có thể tạo Controllers một cách dễ dàng bằng cách thêm một lớp mới trong thư mục `Controllers` và kế thừa từ lớp `ControllerBase` hoặc `Controller`. `ControllerBase` là cơ sở cho tất cả các Controllers API trong ASP.NET Core, trong khi `Controller` cung cấp thêm các phương thức hữu ích cho các ứng dụng MVC truyền thống.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly IProductService _productService;

        public ProductsController(IProductService productService)
        {
            _productService = productService;
        }

        [HttpGet]
        public IActionResult GetProducts()
        {
            var products = _productService.GetAllProducts();
            return Ok(products);
        }

        [HttpGet("{id}")]
        public IActionResult GetProduct(int id)
        {
            var product = _productService.GetProductById(id);
            if (product == null)
            {
                return NotFound();
            }
            return Ok(product);
        }

        [HttpPost]
        public IActionResult CreateProduct([FromBody] Product product)
        {
            _productService.AddProduct(product);
            return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
        }

        [HttpPut("{id}")]
        public IActionResult UpdateProduct(int id, [FromBody] Product updatedProduct)
        {
            if (id != updatedProduct.Id)
            {
                return BadRequest();
            }
            _productService.UpdateProduct(updatedProduct);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public IActionResult DeleteProduct(int id)
        {
            _productService.DeleteProduct(id);
            return NoContent();
        }
    }
}
```
Trong ví dụ trên:

- `[ApiController]`: Chỉ định rằng đây là một API controller và ASP.NET Core sẽ tự động thực hiện một số hành động như xác thực model state.
- `[Route("api/[controller]")]`: Định nghĩa đường dẫn URL cơ bản cho các hành động trong controller này. `[controller]` sẽ được thay thế bằng tên controller (trong trường hợp này là `Products`).
- `IActionResult`: Là kiểu trả về tiêu chuẩn cho các phương thức trong controller, cho phép trả về các loại kết quả khác nhau như `Ok`, `NotFound`, `BadRequest`, v.v.
## 3. Giới Thiệu Về REST API
REST (Representational State Transfer) là một kiến trúc web dịch vụ phổ biến dựa trên HTTP. Một API tuân theo REST thường được gọi là RESTful API. Các thành phần chính của REST API bao gồm:

- Resource: Là đối tượng chính mà API quản lý, chẳng hạn như `Product`, `User`, `Order`.
- URI: Định danh cho các resources, thường là đường dẫn URL như `/api/products` để truy cập danh sách sản phẩm.
- HTTP Verbs: Chỉ định hành động thực hiện trên resource, bao gồm GET, POST, PUT, DELETE.
- Stateless: Mỗi yêu cầu từ client tới server phải chứa đủ thông tin để server có thể xử lý yêu cầu mà không cần dựa vào trạng thái được lưu từ các yêu cầu trước.
## 4. Các HTTP Verbs Phổ Biến Trong REST API
- GET: Dùng để truy vấn thông tin. Ví dụ: `GET /api/products` để lấy danh sách sản phẩm.
- POST: Dùng để tạo mới một resource. Ví dụ: `POST /api/products` để tạo mới một sản phẩm.
- PUT: Dùng để cập nhật toàn bộ một resource. Ví dụ: `PUT /api/products/1` để cập nhật thông tin sản phẩm có ID là 1.
- DELETE: Dùng để xóa một resource. Ví dụ: `DELETE /api/products/1` để xóa sản phẩm có ID là 1.
- PATCH: Dùng để cập nhật một phần của resource. Ví dụ: `PATCH /api/products/1` để cập nhật một số thuộc tính của sản phẩm có ID là 1.
## 5. Triển Khai REST API Trong ASP.NET Core
Triển khai REST API trong ASP.NET Core khá đơn giản nhờ vào các thành phần như Controllers, Models, và Services. Dưới đây là các bước cơ bản:

Bước 1: Tạo Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Description { get; set; }
}
```
Bước 2: Tạo Service

Service sẽ chịu trách nhiệm xử lý logic kinh doanh liên quan đến dữ liệu.

```csharp
public interface IProductService
{
    IEnumerable<Product> GetAllProducts();
    Product GetProductById(int id);
    void AddProduct(Product product);
    void UpdateProduct(Product product);
    void DeleteProduct(int id);
}

public class ProductService : IProductService
{
    private readonly List<Product> _products = new List<Product>();

    public IEnumerable<Product> GetAllProducts() => _products;

    public Product GetProductById(int id) => _products.FirstOrDefault(p => p.Id == id);

    public void AddProduct(Product product) => _products.Add(product);

    public void UpdateProduct(Product product)
    {
        var existingProduct = GetProductById(product.Id);
        if (existingProduct != null)
        {
            existingProduct.Name = product.Name;
            existingProduct.Price = product.Price;
            existingProduct.Description = product.Description;
        }
    }

    public void DeleteProduct(int id) => _products.RemoveAll(p => p.Id == id);
}
```
Bước 3: Tạo Controller

Controller sẽ chịu trách nhiệm nhận yêu cầu từ client, gọi các phương thức từ service, và trả về kết quả.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public IActionResult GetProducts()
    {
        var products = _productService.GetAllProducts();
        return Ok(products);
    }

    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        var product = _productService.GetProductById(id);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }

    [HttpPost]
    public IActionResult CreateProduct([FromBody] Product product)
    {
        _productService.AddProduct(product);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }

    [HttpPut("{id}")]
    public IActionResult UpdateProduct(int id, [FromBody] Product updatedProduct)
    {
        if (id != updatedProduct.Id)
        {
            return BadRequest();
        }
        _productService.UpdateProduct(updatedProduct);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public IActionResult DeleteProduct(int id)
    {
        _productService.DeleteProduct(id);
        return NoContent();
    }
}
```
## 6. Các Phương Pháp Kiểm Tra REST API
Bạn có thể kiểm tra REST API bằng cách sử dụng các công cụ như Postman, Swagger, hoặc đơn giản chỉ cần sử dụng trình duyệt để thực hiện các yêu cầu GET.

Swagger

Swagger là một công cụ mạnh mẽ trong ASP.NET Core, giúp bạn tự động tạo tài liệu API và cung cấp giao diện người dùng để kiểm tra các điểm cuối API.

```csharp
// Trong file Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddSwaggerGen();

var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```
Khi chạy ứng dụng, bạn có thể truy cập vào `https://localhost:5001/swagger/index.html` để kiểm tra API.

7. Tổng kết
Controllers và REST API là những khối xây dựng cơ bản của ứng dụng ASP.NET Core. Bằng cách hiểu cách tạo và triển khai các controllers, bạn có thể xây dựng các ứng dụng web mạnh mẽ và có khả năng mở rộng. REST API cung cấp một cách tiếp cận tiêu chuẩn để giao tiếp giữa client và server, và ASP.NET Core cung cấp các công cụ mạnh mẽ để triển khai chúng một cách hiệu quả.
