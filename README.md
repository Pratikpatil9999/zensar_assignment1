 [HttpPost("upload")]
        public async Task<IActionResult> Upload(IFormFile file, [FromServices] EmpDbContext dbContext)
        {
            try
            {
                using (var stream = new MemoryStream())
                {
                    await file.CopyToAsync(stream);
                    ExcelPackage.LicenseContext = LicenseContext.NonCommercial;


                    using (var package = new ExcelPackage(stream))
                    {
                        var worksheet = package.Workbook.Worksheets[0];
                        int total = worksheet.Dimension.End.Row;
                        var employees = new List<EmpModel>();

                        int firstRow = worksheet.Dimension.Start.Row;
                        int lastRow = worksheet.Dimension.End.Row;
                        int firstCol = worksheet.Dimension.Start.Column;
                        int lastCol = worksheet.Dimension.End.Column;
                        Console.WriteLine("firstrow" + firstRow + " firstcol " + firstCol);

                        for (int row = firstRow+1; row <= lastRow; row++)
                        {

                            if (worksheet.Cells[row, firstCol].Value == null || string.IsNullOrEmpty(worksheet.Cells[row, firstCol].Value.ToString()))
                            {
                                continue;
                            }
                            var id = int.Parse(worksheet.Cells[row, firstCol].Value.ToString());
                            var name = worksheet.Cells[row, firstCol+1].Value.ToString();

                            var employee = new EmpModel
                            {
                                id = id,
                                Name = name
                            };

                           

                            employees.Add(employee);
                        }
                        dbContext.my_table.AddRange(employees);
                        await dbContext.SaveChangesAsync();
                    }
                }

                return Ok();
            }
            catch (Exception ex)
            {
                return BadRequest(ex.Message);
            }
        }


    }
}
update this code snippet so that it can update data if data with id is already exist or add new data else. just don't add data one by one
