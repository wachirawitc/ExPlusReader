Supporter extension for EPPlus to read and parse value in cell with condition.

**Basic**<br />
For example value in row 1 and column 1 is "1"
```csharp
            using (var package = new ExcelPackage(excelfile))
            {
                var worksheet = package.Workbook.Worksheets[1];

                int value = worksheet.Cells[1, 1]
                    .Cast<int>()
                    .NumericOnly()
                        .WithMessage(cell => $"{cell.Address} is support numeric only")
                    .Get();

            }
```

<br />

**Custom convert value**<br />
For example value in row 1 and column 1 is "21/04/2016"
```csharp
            using (var package = new ExcelPackage(excelfile))
            {
                var worksheet = package.Workbook.Worksheets[1];

                DateTime? value = worksheet.Cells[1, 1]
                            .Cast(value => ToDateTime(value))
                            .NotNull()
                            .Get();

            }
            
            private static DateTime? ToDateTime(object value)
            {
                        string[] formats = { "dd/MM/yyyy" };
                        DateTime outTime;
                        var isSuccess = DateTime.TryParseExact(
                                    value.ToString(), 
                                    formats, 
                                    CultureInfo.InvariantCulture, 
                                    DateTimeStyles.None, 
                                    out outTime);
                        return isSuccess == false ? (DateTime?)null : outTime;
            }
            
```

On special convert please implement **IConverter** interface.<br />
```csharp
    public class NullableDateTimeConverter : IConverter<DateTime?>
    {
        private readonly object value;
        private readonly List<string> formats;

        public NullableDateTimeConverter(object value, List<string> formats)
        {
            this.value = value;
            this.formats = formats;
        }

        public DateTime? Get()
        {
            if (value == null)
            {
                return null;
            }

            DateTime outTime;
            var isSuccess = DateTime.TryParseExact(value.ToString(),
                formats.ToArray(),
                CultureInfo.InvariantCulture,
                DateTimeStyles.None,
                out outTime);

            return isSuccess == false ? (DateTime?)null : outTime;
        }
    }
```
When you use
```csharp
            var supportDateFormat = new List<string> { "dd/MM/yyyy" };
            using (var package = new ExcelPackage(fileInfo))
            {
                var worksheet = package.Workbook.Worksheets[1];

                var actual = worksheet.Cells[1, 1]
                    .Cast<DateTime?>(value => new NullableDateTimeConverter(value, supportDateFormat))
                    .Get();
            }
```

