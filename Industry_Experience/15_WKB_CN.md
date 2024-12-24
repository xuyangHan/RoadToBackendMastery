# 从理论到实践：WKB 在 C# 和 PostGIS 中的完整应用指南

## 什么是WKB？

**知名二进制(Well-Known Binary, WKB)** 是一种用于表示几何对象的二进制格式。它用于以紧凑的二进制形式传输和存储几何数据。WKB广泛应用于地理信息系统（GIS），尤其适用于像PostGIS这样的空间数据库。

WKB格式按照特定顺序结构化字节数组：

1. **头部:** 表示数据的字节顺序（字节序）。
2. **类型:** 指定几何类型（例如，点、线字符串、多边形）。
3. **坐标:** 包含几何对象的坐标。

## 在PostGIS数据库中的使用

**PostGIS**是PostgreSQL的空间数据库扩展。它增加了对地理对象的支持，允许在SQL中运行位置查询。PostGIS使用WKB以紧凑的二进制格式存储几何数据，从而提高存储和检索的效率。

**使用WKB的优势:**

- **效率:** WKB比文本格式（如知名文本格式WKT）更紧凑，节省存储空间并提高性能。
- **精度:** 二进制格式避免了在文本和二进制表示之间转换时可能出现的精度损失。
- **互操作性:** WKB是一种被许多GIS应用程序和库支持的标准格式，确保兼容性。您可以直接使用GIS工具查看这些对象。

## C# 实现

要在C#中使用WKB，您可以创建一个`WKBConverter`类。该类将处理几何对象到WKB格式的转换。以下是一个简单的实现：

1. **定义几何类型和WKB转换器:**

    首先定义几何对象：

    ```csharp
    public enum WKBType
    {
            Point2D = 1,
            PointM = 2,
            LineStringM = 3,
            Polygon2D = 4,
    }

    public class PointM 
    {
        public double X { get; set; }
        public double Y { get; set; }
        public double M { get; set; } 

        public PointM() { }

        public PointM(double x, double y, double m)
        {
            X = x; Y = y; M = m;
        }
    }

    public class LineStringM : List<PointM>
    { }

    public class Polygon2D : List<PointM>
    { }
    ```

    然后准备`WKBConverter`类：

    ```csharp
    public class WKBConverter
    {
        private const int HeaderLength = 5;
        private const int DoubleLength = 8;
        private const int IntLength = 4;
        private int currIdx = 0;

        private void InitWKB(ref byte[] arr, WKBType type)
        {
            AddEndian(ref arr);
            AddWKBType(ref arr, type);
        }

        private void AddEndian(ref byte[] arr)
        {
            arr[currIdx++] = BitConverter.IsLittleEndian ? (byte)1 : (byte)0;
        }

        private void AddWKBType(ref byte[] arr, WKBType type)
        {
            byte[] geomType = BitConverter.GetBytes((int)type);
            Array.Copy(geomType, 0, arr, currIdx, IntLength);
            currIdx += IntLength;
        }

        private void AddDouble(ref byte[] arr, double val)
        {
            byte[] bytes = BitConverter.GetBytes(val);
            Array.Copy(bytes, 0, arr, currIdx, DoubleLength);
            currIdx += DoubleLength;
        }

        private void AddInt(ref byte[] arr, int val)
        {
            byte[] bytes = BitConverter.GetBytes(val);
            Array.Copy(bytes, 0, arr, currIdx, IntLength);
            currIdx += IntLength;
        }

        private double ReadDouble(byte[] arr)
        {
            double val = BitConverter.ToDouble(arr, currIdx);
            currIdx += DoubleLength;
            return val;
        }

        private int ReadInt(byte[] arr)
        {
            int val = BitConverter.ToInt32(arr, currIdx);
            currIdx += IntLength;
            return val;
        }

        private void ResetIdx()
        {
            currIdx = 0;
        }

        public byte[] ToBytes(PointM point)
        {
            ResetIdx();

            byte[] arr = new byte[HeaderLength + DoubleLength * 3];
            InitWKB(ref arr, WKBType.PointM);
            AddDouble(ref arr, point.X);
            AddDouble(ref arr, point.Y);
            AddDouble(ref arr, point.M);

            ResetIdx();
            return arr;
        }

        public LineStringM ToLineStringM(byte[] arr)
        {
            ResetIdx();

            try
            {
                if (!IsValid(arr, WKBType.LineStringM))
                {
                    return null;
                }
            }
            catch (Exception)
            {
                return null;
            }

            int n = ReadInt(arr); // number of points

            LineStringM linestring = new LineStringM();
            for (int i = 0; i < n; i++)
            {
                PointM p = new PointM();
                try
                {
                    p.X = ReadDouble(arr);
                    p.Y = ReadDouble(arr);
                    p.M = ReadDouble(arr);
                    linestring.Add(p);
                }
                catch
                {
                    Console.WriteLine("Error reading the Point. ");
                }
            }

            ResetIdx();

            return linestring;
        }

        private bool IsValid(byte[] arr, WKBType type)
        {
            if (arr == null || arr.Length == 0) return false;
            if (arr[currIdx++] == 0)
            {
                throw new Exception("Data is 'big endian' and this converter is designed for 'little endian' data.");
            }

            WKBType dataType = (WKBType)BitConverter.ToUInt32(arr, currIdx);
            currIdx += IntLength;
            return type == dataType;
        }
    }
    ```

2. **使用示例:**

    ```csharp
    LineStringM linestring = new LineStringM();
    foreach (var pos in trajectory.Positions)
    {
        linestring.Add(new PointM()
        {
            X = pos.Longitude,
            Y = pos.Latitude,
            M = pos.unix_s,
        });
    }

    // convert linestring to byte[]
    WKBConverter wKBConverter = new WKBConverter();
    byte[] wkb = wKBConverter.ToBytes(linestring);

    // convert byte[] data back to LineStringM
    LineStringM points = wKBConverter.ToLineStringM(wkb); 
    ```

## 从数据库保存和读取数据

1. **保存数据到数据库:**

    ```csharp
    string sql = "INSERT INTO your_table (boundary) VALUES (ST_GeomFromWKB(@boundary))";
    
    Polygon2D poly = new Polygon2D(); // populate it 
    WKBConverter wKBConverter = new WKBConverter();
    byte[] boundary = wKBConverter.ToBytes(poly);

    var param = new { boundary = boundary };
    PostgreSql.Execute(sql, param); // check previous stories on how to set up dapper 
    ```

2. **从数据库读取数据:**

    ```csharp
    string sql = @"SELECT ST_AsBinary(boundary) AS wkb From your_table";
    List<boundary> boundaries = PostgreSql.Query<boundary>(sql).ToList(); // check previous stories on how to set up dapper
    ```

## 需要注意的关键事项

1. **字节顺序:** 确保字节顺序（字节序）在系统和WKB格式之间匹配。
2. **线程安全:** 如果您的转换器方法在多线程环境中使用，确保每个线程使用自己的`WKBConverter`实例以避免并发问题。
3. **数据验证:** 在处理之前始终验证WKB数据，以避免因数据损坏或意外数据导致的错误。
4. **错误处理:** 实施健全的错误处理，以管理数据格式意外或损坏的情况。

通过遵循这些指南并利用`WKBConverter`类，您可以在C#应用程序中有效地管理WKB数据，确保与PostGIS支持的数据库无缝集成。
