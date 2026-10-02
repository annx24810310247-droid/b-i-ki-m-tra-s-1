using System;
using System.Collections.Generic;
using System.Linq;
namespace AutoSpeedLogistics
{
    // ==========================================
    // A. ABSTRACT CLASS: PhuongTien
    // ==========================================
    public abstract class PhuongTien
    {
        // Private Fields
        private string _maPT;
        private string _tenHang;
        private int _namSanXuat;
        private decimal _giaGoc;
        // Encapsulated Properties with Validation
        public string MaPT
        {
            get => _maPT;
            set => _maPT = string.IsNullOrWhiteSpace(value) ? "PT000" : value.Trim();
        }

        public string TenHang
        {
            get => _tenHang;
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                    throw new ArgumentException("Tên hãng không được để trống!");
                _tenHang = value.Trim();
            }
        }

        public int NamSanXuat
        {
            get => _namSanXuat;
            set
            {
                int currentYear = DateTime.Now.Year;
                if (value < 1900 || value > currentYear)
                    throw new ArgumentException($"Năm sản xuất phải từ 1900 đến {currentYear}!");
                _namSanXuat = value;
            }
        }

        public decimal GiaGoc
        {
            get => _giaGoc;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Giá gốc phải lớn hơn 0!");
                _giaGoc = value;
            }
        }

        // Constructor
        protected PhuongTien(string maPT, string tenHang, int namSanXuat, decimal giaGoc)
        {
            MaPT = maPT;
            TenHang = tenHang;
            NamSanXuat = namSanXuat;
            GiaGoc = giaGoc;
        }

        // Abstract Method (Polymorphism)
        public abstract decimal TinhGiaLanBanh();

        // Virtual Method
        public virtual string GetInfo()
        {
            return $"[Mã: {MaPT}] Hãng: {TenHang} | Năm SX: {NamSanXuat} | Giá gốc: {GiaGoc:N0} VNĐ";
        }
    }

    // ==========================================
    // B. CLASS: OTo (Inherits PhuongTien)
    // ==========================================
    public class OTo : PhuongTien
    {
        private int _soChoNgoi;
        private double _dungTichDongCo;

        public int SoChoNgoi
        {
            get => _soChoNgoi;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Số chỗ ngồi phải lớn hơn 0!");
                _soChoNgoi = value;
            }
        }

        public double DungTichDongCo
        {
            get => _dungTichDongCo;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Dung tích động cơ phải lớn hơn 0!");
                _dungTichDongCo = value;
            }
        }

        public OTo(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int soChoNgoi, double dungTichDongCo)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            SoChoNgoi = soChoNgoi;
            DungTichDongCo = dungTichDongCo;
        }

        public override decimal TinhGiaLanBanh()
        {
            if (SoChoNgoi <= 9)
            {
                // Giá lăn bánh = Giá gốc + Lệ phí trước bạ (12%) + Thuế Tiêu thụ đặc biệt (30%)
                decimal thueTruocBa = GiaGoc * 0.12m;
                decimal thueTTDB = GiaGoc * 0.30m;
                return GiaGoc + thueTruocBa + thueTTDB;
            }
            else
            {
                // Giá lăn bánh = Giá gốc + Lệ phí trước bạ (10%)
                decimal thueTruocBa = GiaGoc * 0.10m;
                return GiaGoc + thueTruocBa;
            }
        }

        public override string GetInfo()
        {
            return $"{base.GetInfo()} | Loại: Ô tô | Số chỗ: {SoChoNgoi} | Dung tích: {DungTichDongCo}L | Giá lăn bánh: {TinhGiaLanBanh():N0} VNĐ";
        }
    }

    // ==========================================
    // C. CLASS: XeMay (Inherits PhuongTien)
    // ==========================================
    public class XeMay : PhuongTien
    {
        private int _dungTichXylanh;

        public int DungTichXylanh
        {
            get => _dungTichXylanh;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Dung tích xilanh phải lớn hơn 0 cc!");
                _dungTichXylanh = value;
            }
        }

        public XeMay(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int dungTichXylanh)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            DungTichXylanh = dungTichXylanh;
        }

        public override decimal TinhGiaLanBanh()
        {
            decimal thueTruocBa = DungTichXylanh < 175 ? GiaGoc * 0.02m : GiaGoc * 0.05m;
            return GiaGoc + thueTruocBa;
        }

        public override string GetInfo()
        {
            return $"{base.GetInfo()} | Loại: Xe máy | Dung tích Xilanh: {DungTichXylanh} cc | Giá lăn bánh: {TinhGiaLanBanh():N0} VNĐ";
        }
    }

    // ==========================================
    // D. CLASS: QuanLyPhuongTien
    // ==========================================
    public class QuanLyPhuongTien
    {
        private readonly List<PhuongTien> _danhSach = new List<PhuongTien>();

        // 1. Thêm phương tiện
        public void AddPhuongTien(PhuongTien pt)
        {
            if (pt != null)
            {
                _danhSach.Add(pt);
            }
        }

        // 2. In toàn bộ danh sách
        public void DisplayAll()
        {
            Console.WriteLine("=================================== DANH SÁCH PHƯƠNG TIỆN ===================================");
            if (_danhSach.Count == 0)
            {
                Console.WriteLine("Danh sách trống!");
                return;
            }

            foreach (var pt in _danhSach)
            {
                Console.WriteLine(pt.GetInfo());
            }
            Console.WriteLine("=============================================================================================\n");
        }

        // 3. Tìm phương tiện có Giá lăn bánh cao nhất (Đa hình)
        public PhuongTien FindMaxGiaLanBanh()
        {
            if (_danhSach.Count == 0) return null;

            return _danhSach.OrderByDescending(pt => pt.TinhGiaLanBanh()).FirstOrDefault();
        }

        // 4. Tìm kiếm theo tên hãng (dùng LINQ)
        public List<PhuongTien> SearchByName(string keyword)
        {
            if (string.IsNullOrWhiteSpace(keyword)) return new List<PhuongTien>();

            return _danhSach
                .Where(pt => pt.TenHang.IndexOf(keyword, StringComparison.OrdinalIgnoreCase) >= 0)
                .ToList();
        }
    }

    // ==========================================
    // MAIN PROGRAM FOR TESTING
    // ==========================================
    class Program
    {
        static void Main(string[] args)
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;
            QuanLyPhuongTien ql = new QuanLyPhuongTien();

            try
            {
                // Thêm dữ liệu mẫu
                ql.AddPhuongTien(new OTo("OT001", "Toyota Camry", 2022, 1050000000m, 5, 2.5));
                ql.AddPhuongTien(new OTo("OT002", "Ford Transit", 2021, 850000000m, 16, 2.2));
                ql.AddPhuongTien(new XeMay("XM001", "Honda Wave Alpha", 2023, 18000000m, 110));
                ql.AddPhuongTien(new XeMay("XM002", "Honda SH 350i", 2022, 150000000m, 350));

                // 1. In toàn bộ
                ql.DisplayAll();

                // 2. Tìm xe có giá lăn bánh cao nhất
                var maxPT = ql.FindMaxGiaLanBanh();
                if (maxPT != null)
                {
                    Console.WriteLine(">>> PHƯƠNG TIỆN CÓ GIÁ LĂN BÁNH CAO NHẤT:");
                    Console.WriteLine(maxPT.GetInfo());
                    Console.WriteLine();
                }

                // 3. Tìm kiếm theo hãng
                string keyword = "Honda";
                Console.WriteLine($">>> KẾT QUẢ TÌM KIẾM THEO HÃNG '{keyword}':");
                var result = ql.SearchByName(keyword);
                foreach (var item in result)
                {
                    Console.WriteLine(item.GetInfo());
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[LỖI DỮ LIỆU]: {ex.Message}");
            }

            Console.ReadLine();
        }
    }

using System;
using System.Collections.Generic;
using System.Linq;
namespace AutoSpeedLogistics
{
    // ==========================================
    // A. ABSTRACT CLASS: PhuongTien
    // ==========================================
    public abstract class PhuongTien
    {
        // Private Fields
        private string _maPT;
        private string _tenHang;
        private int _namSanXuat;
        private decimal _giaGoc;
        // Encapsulated Properties with Validation
        public string MaPT
        {
            get => _maPT;
            set => _maPT = string.IsNullOrWhiteSpace(value) ? "PT000" : value.Trim();
        }

        public string TenHang
        {
            get => _tenHang;
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                    throw new ArgumentException("Tên hãng không được để trống!");
                _tenHang = value.Trim();
            }
        }

        public int NamSanXuat
        {
            get => _namSanXuat;
            set
            {
                int currentYear = DateTime.Now.Year;
                if (value < 1900 || value > currentYear)
                    throw new ArgumentException($"Năm sản xuất phải từ 1900 đến {currentYear}!");
                _namSanXuat = value;
            }
        }

        public decimal GiaGoc
        {
            get => _giaGoc;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Giá gốc phải lớn hơn 0!");
                _giaGoc = value;
            }
        }

        // Constructor
        protected PhuongTien(string maPT, string tenHang, int namSanXuat, decimal giaGoc)
        {
            MaPT = maPT;
            TenHang = tenHang;
            NamSanXuat = namSanXuat;
            GiaGoc = giaGoc;
        }

        // Abstract Method (Polymorphism)
        public abstract decimal TinhGiaLanBanh();

        // Virtual Method
        public virtual string GetInfo()
        {
            return $"[Mã: {MaPT}] Hãng: {TenHang} | Năm SX: {NamSanXuat} | Giá gốc: {GiaGoc:N0} VNĐ";
        }
    }

    // ==========================================
    // B. CLASS: OTo (Inherits PhuongTien)
    // ==========================================
    public class OTo : PhuongTien
    {
        private int _soChoNgoi;
        private double _dungTichDongCo;

        public int SoChoNgoi
        {
            get => _soChoNgoi;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Số chỗ ngồi phải lớn hơn 0!");
                _soChoNgoi = value;
            }
        }

        public double DungTichDongCo
        {
            get => _dungTichDongCo;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Dung tích động cơ phải lớn hơn 0!");
                _dungTichDongCo = value;
            }
        }

        public OTo(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int soChoNgoi, double dungTichDongCo)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            SoChoNgoi = soChoNgoi;
            DungTichDongCo = dungTichDongCo;
        }

        public override decimal TinhGiaLanBanh()
        {
            if (SoChoNgoi <= 9)
            {
                // Giá lăn bánh = Giá gốc + Lệ phí trước bạ (12%) + Thuế Tiêu thụ đặc biệt (30%)
                decimal thueTruocBa = GiaGoc * 0.12m;
                decimal thueTTDB = GiaGoc * 0.30m;
                return GiaGoc + thueTruocBa + thueTTDB;
            }
            else
            {
                // Giá lăn bánh = Giá gốc + Lệ phí trước bạ (10%)
                decimal thueTruocBa = GiaGoc * 0.10m;
                return GiaGoc + thueTruocBa;
            }
        }

        public override string GetInfo()
        {
            return $"{base.GetInfo()} | Loại: Ô tô | Số chỗ: {SoChoNgoi} | Dung tích: {DungTichDongCo}L | Giá lăn bánh: {TinhGiaLanBanh():N0} VNĐ";
        }
    }

    // ==========================================
    // C. CLASS: XeMay (Inherits PhuongTien)
    // ==========================================
    public class XeMay : PhuongTien
    {
        private int _dungTichXylanh;

        public int DungTichXylanh
        {
            get => _dungTichXylanh;
            set
            {
                if (value <= 0)
                    throw new ArgumentException("Dung tích xilanh phải lớn hơn 0 cc!");
                _dungTichXylanh = value;
            }
        }

        public XeMay(string maPT, string tenHang, int namSanXuat, decimal giaGoc, int dungTichXylanh)
            : base(maPT, tenHang, namSanXuat, giaGoc)
        {
            DungTichXylanh = dungTichXylanh;
        }

        public override decimal TinhGiaLanBanh()
        {
            decimal thueTruocBa = DungTichXylanh < 175 ? GiaGoc * 0.02m : GiaGoc * 0.05m;
            return GiaGoc + thueTruocBa;
        }

        public override string GetInfo()
        {
            return $"{base.GetInfo()} | Loại: Xe máy | Dung tích Xilanh: {DungTichXylanh} cc | Giá lăn bánh: {TinhGiaLanBanh():N0} VNĐ";
        }
    }

    // ==========================================
    // D. CLASS: QuanLyPhuongTien
    // ==========================================
    public class QuanLyPhuongTien
    {
        private readonly List<PhuongTien> _danhSach = new List<PhuongTien>();

        // 1. Thêm phương tiện
        public void AddPhuongTien(PhuongTien pt)
        {
            if (pt != null)
            {
                _danhSach.Add(pt);
            }
        }

        // 2. In toàn bộ danh sách
        public void DisplayAll()
        {
            Console.WriteLine("=================================== DANH SÁCH PHƯƠNG TIỆN ===================================");
            if (_danhSach.Count == 0)
            {
                Console.WriteLine("Danh sách trống!");
                return;
            }

            foreach (var pt in _danhSach)
            {
                Console.WriteLine(pt.GetInfo());
            }
            Console.WriteLine("=============================================================================================\n");
        }

        // 3. Tìm phương tiện có Giá lăn bánh cao nhất (Đa hình)
        public PhuongTien FindMaxGiaLanBanh()
        {
            if (_danhSach.Count == 0) return null;

            return _danhSach.OrderByDescending(pt => pt.TinhGiaLanBanh()).FirstOrDefault();
        }

        // 4. Tìm kiếm theo tên hãng (dùng LINQ)
        public List<PhuongTien> SearchByName(string keyword)
        {
            if (string.IsNullOrWhiteSpace(keyword)) return new List<PhuongTien>();

            return _danhSach
                .Where(pt => pt.TenHang.IndexOf(keyword, StringComparison.OrdinalIgnoreCase) >= 0)
                .ToList();
        }
    }

    // ==========================================
    // MAIN PROGRAM FOR TESTING
    // ==========================================
    class Program
    {
        static void Main(string[] args)
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;
            QuanLyPhuongTien ql = new QuanLyPhuongTien();

            try
            {
                // Thêm dữ liệu mẫu
                ql.AddPhuongTien(new OTo("OT001", "Toyota Camry", 2022, 1050000000m, 5, 2.5));
                ql.AddPhuongTien(new OTo("OT002", "Ford Transit", 2021, 850000000m, 16, 2.2));
                ql.AddPhuongTien(new XeMay("XM001", "Honda Wave Alpha", 2023, 18000000m, 110));
                ql.AddPhuongTien(new XeMay("XM002", "Honda SH 350i", 2022, 150000000m, 350));

                // 1. In toàn bộ
                ql.DisplayAll();

                // 2. Tìm xe có giá lăn bánh cao nhất
                var maxPT = ql.FindMaxGiaLanBanh();
                if (maxPT != null)
                {
                    Console.WriteLine(">>> PHƯƠNG TIỆN CÓ GIÁ LĂN BÁNH CAO NHẤT:");
                    Console.WriteLine(maxPT.GetInfo());
                    Console.WriteLine();
                }

                // 3. Tìm kiếm theo hãng
                string keyword = "Honda";
                Console.WriteLine($">>> KẾT QUẢ TÌM KIẾM THEO HÃNG '{keyword}':");
                var result = ql.SearchByName(keyword);
                foreach (var item in result)
                {
                    Console.WriteLine(item.GetInfo());
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[LỖI DỮ LIỆU]: {ex.Message}");
            }

            Console.ReadLine();
        }
    }


}

}
<img width="1162" height="584" alt="image" src="https://github.com/user-attachments/assets/c5a7177a-42f6-408a-980f-f9922d3673fd" />
