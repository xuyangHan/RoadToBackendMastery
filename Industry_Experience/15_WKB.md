# From Theory to Practice: A Complete Guide to WKB in C# and PostGIS

## What is WKB?

**Well-Known Binary (WKB)** is a binary format used to represent geometric objects. It is used to transmit and store geometric data in a compact binary form. WKB is widely used in Geographic Information Systems (GIS), especially in spatial databases like PostGIS.

The WKB format is structured as a byte array in a specific order:

1. **Header:** Represents the byte order (endianness).
2. **Type:** Specifies the geometry type (e.g., point, linestring, polygon).
3. **Coordinates:** Contains the coordinates of the geometry object.

## Usage in PostGIS Database

**PostGIS** is a spatial database extension for PostgreSQL. It adds support for geographic objects and allows spatial queries in SQL. PostGIS uses WKB to store geometric data in a compact binary format, enhancing storage and retrieval efficiency.

**Advantages of Using WKB:**

- **Efficiency:** WKB is more compact than text formats like Well-Known Text (WKT), saving storage space and improving performance.
- **Precision:** The binary format avoids precision loss that can occur when converting between text and binary representations.
- **Interoperability:** WKB is a standard format supported by many GIS applications and libraries, ensuring compatibility. You can directly view these objects with GIS tools.

## C# Implementation

To use WKB in C#, you can create a `WKBConverter` class. This class will handle the conversion of geometric objects to WKB format. Below is a simple implementation:

1. **Define Geometry Types and WKB Converter:**

    First, define the geometry objects:

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

    Then prepare the `WKBConverter` class:

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

2. **Example Usage:**

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

    // Convert linestring to byte[]
    WKBConverter wKBConverter = new WKBConverter();
    byte[] wkb = wKBConverter.ToBytes(linestring);

    // Convert byte[] data back to LineStringM
    LineStringM points = wKBConverter.ToLineStringM(wkb); 
    ```

## Saving and Retrieving Data from the Database

1. **Saving Data to the Database:**

    ```csharp
    string sql = "INSERT INTO your_table (boundary) VALUES (ST_GeomFromWKB(@boundary))";
    
    Polygon2D poly = new Polygon2D(); // populate it 
    WKBConverter wKBConverter = new WKBConverter();
    byte[] boundary = wKBConverter.ToBytes(poly);

    var param = new { boundary = boundary };
    PostgreSql.Execute(sql, param); // check previous stories on how to set up dapper 
    ```

2. **Retrieving Data from the Database:**

    ```csharp
    string sql = @"SELECT ST_AsBinary(boundary) AS wkb From your_table";
    List<boundary> boundaries = PostgreSql.Query<boundary>(sql).ToList(); // check previous stories on how to set up dapper
    ```

## Key Considerations

1. **Byte Order:** Ensure that the byte order (endianness) matches between your system and the WKB format.
2. **Thread Safety:** If your converter methods are used in a multithreaded environment, make sure each thread uses its own `WKBConverter` instance to avoid concurrency issues.
3. **Data Validation:** Always validate WKB data before processing it to avoid errors due to corrupted or unexpected data.
4. **Error Handling:** Implement robust error handling to manage unexpected data formats or corrupt data.

By following these guidelines and using the `WKBConverter` class, you can efficiently manage WKB data in your C# applications and ensure seamless integration with PostGIS-enabled databases.
