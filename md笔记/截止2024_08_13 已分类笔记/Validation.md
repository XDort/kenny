#### FluentValidation验证数据输入

```c#
public class CustomerValidator : AbstractValidator<Customer>
{
  public CustomerValidator()
  {
    RuleFor(x => x.Surname).NotEmpty();
    RuleFor(x => x.Forename).NotEmpty().WithMessage("Please specify a first name");
    RuleFor(x => x.Discount).NotEqual(0).When(x => x.HasDiscount);
    RuleFor(x => x.Address).Length(20, 250);
    RuleFor(x => x.Postcode).Must(BeAValidPostcode).WithMessage("Please specify a valid postcode");
  }

  private bool BeAValidPostcode(string postcode)
  {
    // custom postcode validating logic goes here
  }
}
```

指定输入数据的泛型比如command、request类

对指定属性进行校验设置，must方法传入func进行校验