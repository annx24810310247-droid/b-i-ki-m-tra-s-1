PROM1
using System;
using System.ComponentModel;
using System.IO;
using System.Linq;
using System.Text;
using System.Windows.Forms;

namespace baikiemtraso01
{
    public partial class Form1 : Form
    {
        private BindingList<Product> _productList = new BindingList<Product>();
        private BindingSource _bindingSource = new BindingSource();
        private BindingList<Category> _categoryList = new BindingList<Category>();

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            InitCategories();
            InitDataBinding();
            RegisterEvents();
            UpdateStatus();
        }

        private void InitCategories()
        {
            _categoryList = new BindingList<Category>
            {
                new Category { Id = 1, Name = "Điện thoại" },
                new Category { Id = 2, Name = "Laptop" },
                new Category { Id = 3, Name = "Phụ kiện" }
            };

            cboCategory.DataSource = _categoryList;
            cboCategory.DisplayMember = "Name";
            cboCategory.ValueMember = "Id";
        }

        private void InitDataBinding()
        {
            // Dữ liệu mẫu ban đầu
            _productList.Add(new Product { Id = "SP01", Name = "iPhone 15 Pro", Price = 28000000, Quantity = 10, CategoryId = 1 });
            _productList.Add(new Product { Id = "SP02", Name = "MacBook Air M2", Price = 24500000, Quantity = 5, CategoryId = 2 });

            _bindingSource.DataSource = _productList;
            dgvProducts.AutoGenerateColumns = false;
            dgvProducts.DataSource = _bindingSource;
        }

        private void RegisterEvents()
        {
            // Gán sự kiện cho các nút chưa có trong Designer
            button3.Click += button3_Click; // Nút Sửa
            button4.Click += button4_Click; // Nút Xóa
            btnChooseImage.Click += btnChooseImage_Click;
            dgvProducts.SelectionChanged += dgvProducts_SelectionChanged;
            exportCSVToolStripMenuItem.Click += exportCSVToolStripMenuItem_Click;
            exitToolStripMenuItem.Click += exitToolStripMenuItem_Click;
        }

        // --- XỬ LÝ ĐỔ DỮ LIỆU TỪ BẢNG LÊN TEXTBOX ---
        private void dgvProducts_SelectionChanged(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow != null && dgvProducts.CurrentRow.DataBoundItem is Product selectedProduct)
            {
                txtProductId.Text = selectedProduct.Id;
                txtProductName.Text = selectedProduct.Name;
                txtUnitPrice.Text = selectedProduct.Price.ToString("G0");
                txtQuantity.Text = selectedProduct.Quantity.ToString();
                cboCategory.SelectedValue = selectedProduct.CategoryId;

                if (!string.IsNullOrEmpty(selectedProduct.ImagePath) && File.Exists(selectedProduct.ImagePath))
                {
                    picAvatar.ImageLocation = selectedProduct.ImagePath;
                }
                else
                {
                    picAvatar.Image = null;
                }
            }
        }

        // --- HÀM KIỂM TRA ĐIỀU KIỆN (VALIDATION) ---
        private bool ValidateInput()
        {
            errorProvider1.Clear();
            bool isValid = true;

            if (string.IsNullOrWhiteSpace(txtProductId.Text))
            {
                errorProvider1.SetError(txtProductId, "Mã sản phẩm không được để trống!");
                isValid = false;
            }

            if (string.IsNullOrWhiteSpace(txtProductName.Text))
            {
                errorProvider1.SetError(txtProductName, "Tên sản phẩm không được để trống!");
                isValid = false;
            }

            if (!decimal.TryParse(txtUnitPrice.Text, out decimal price) || price <= 0)
            {
                errorProvider1.SetError(txtUnitPrice, "Đơn giá phải là số > 0!");
                isValid = false;
            }

            if (!int.TryParse(txtQuantity.Text, out int qty) || qty < 0)
            {
                errorProvider1.SetError(txtQuantity, "Số lượng phải là số nguyên >= 0!");
                isValid = false;
            }

            return isValid;
        }

        // --- NÚT THÊM (button2) ---
        private void button2_Click(object sender, EventArgs e)
        {
            if (!ValidateInput()) return;

            string id = txtProductId.Text.Trim();
            if (_productList.Any(p => p.Id.Equals(id, StringComparison.OrdinalIgnoreCase)))
            {
                errorProvider1.SetError(txtProductId, "Mã sản phẩm đã tồn tại!");
                return;
            }

            Product newProd = new Product
            {
                Id = id,
                Name = txtProductName.Text.Trim(),
                Price = decimal.Parse(txtUnitPrice.Text),
                Quantity = int.Parse(txtQuantity.Text),
                CategoryId = Convert.ToInt32(cboCategory.SelectedValue),
                ImagePath = picAvatar.ImageLocation ?? string.Empty
            };

            _productList.Add(newProd);
            UpdateStatus();
            MessageBox.Show("Thêm sản phẩm thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }

        // --- NÚT SỬA (button3) ---
        private void button3_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow?.DataBoundItem is Product selectedProduct)
            {
                if (!ValidateInput()) return;

                selectedProduct.Name = txtProductName.Text.Trim();
                selectedProduct.Price = decimal.Parse(txtUnitPrice.Text);
                selectedProduct.Quantity = int.Parse(txtQuantity.Text);
                selectedProduct.CategoryId = Convert.ToInt32(cboCategory.SelectedValue);
                selectedProduct.ImagePath = picAvatar.ImageLocation ?? string.Empty;

                _bindingSource.ResetBindings(false);
                MessageBox.Show("Cập nhật thành công!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        // --- NÚT XÓA (button4) ---
        private void button4_Click(object sender, EventArgs e)
        {
            if (dgvProducts.CurrentRow?.DataBoundItem is Product selectedProduct)
            {
                var confirm = MessageBox.Show($"Bạn có chắc muốn xóa {selectedProduct.Name}?",
                    "Xác nhận", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

                if (confirm == DialogResult.Yes)
                {
                    _productList.Remove(selectedProduct);
                    UpdateStatus();
                }
            }
        }

        // --- NÚT CHỌN ẢNH ---
        private void btnChooseImage_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog ofd = new OpenFileDialog())
            {
                ofd.Filter = "Hình ảnh (*.jpg; *.png; *.jpeg)|*.jpg;*.png;*.jpeg";
                if (ofd.ShowDialog() == DialogResult.OK)
                {
                    picAvatar.ImageLocation = ofd.FileName;
                }
            }
        }

        // --- XUẤT CSV ---
        private void exportCSVToolStripMenuItem_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog sfd = new SaveFileDialog())
            {
                sfd.Filter = "CSV File (*.csv)|*.csv";
                sfd.FileName = "DanhSach.csv";
                if (sfd.ShowDialog() == DialogResult.OK)
                {
                    StringBuilder sb = new StringBuilder();
                    sb.AppendLine("Mã SP,Tên SP,Đơn Giá,Số Lượng,Mã Danh Mục");
                    foreach (var p in _productList)
                    {
                        sb.AppendLine($"\"{p.Id}\",\"{p.Name}\",{p.Price},{p.Quantity},{p.CategoryId}");
                    }
                    File.WriteAllText(sfd.FileName, sb.ToString(), Encoding.UTF8);
                    MessageBox.Show("Xuất file thành công!", "Thông báo");
                }
            }
        }

        // --- THOÁT ---
        private void exitToolStripMenuItem_Click(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void UpdateStatus()
        {
            lblStatus.Text = $"Tổng số sản phẩm: {_productList.Count}";
        }

        // =========================================================================
        // CÁC HÀM TRỐNG ĐỂ CHỐNG LỖI DO VÔ TÌNH CLICK ĐÚP TRONG GIAO DIỆN DESIGNER
        // =========================================================================
        private void tableLayoutPanel1_Paint(object sender, PaintEventArgs e) { }
        private void label1_Click(object sender, EventArgs e) { }
        private void label3_Click(object sender, EventArgs e) { }
        private void dgvProducts_CellContentClick(object sender, DataGridViewCellEventArgs e) { }
    }

    // --- LỚP DỮ LIỆU ĐI KÈM CẦN THIẾT ---
    public class Category
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
    }

    public class Product
    {
        public string Id { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public int Quantity { get; set; }
        public int CategoryId { get; set; }
        public string ImagePath { get; set; } = string.Empty;
    }
}
PROM DEGIN
namespace baikiemtraso01
{
    partial class Form1
    {
        /// <summary>
        ///  Required designer variable.
        /// </summary>
        private System.ComponentModel.IContainer components = null;

        /// <summary>
        ///  Clean up any resources being used.
        /// </summary>
        /// <param name="disposing">true if managed resources should be disposed; otherwise, false.</param>
        protected override void Dispose(bool disposing)
        {
            if (disposing && (components != null))
            {
                components.Dispose();
            }
            base.Dispose(disposing);
        }

        #region Windows Form Designer generated code

        /// <summary>
        ///  Required method for Designer support - do not modify
        ///  the contents of this method with the code editor.
        /// </summary>
        private void InitializeComponent()
        {
            components = new System.ComponentModel.Container();
            DataGridViewCellStyle dataGridViewCellStyle1 = new DataGridViewCellStyle();
            tableLayoutPanel1 = new TableLayoutPanel();
            panel1 = new Panel();
            label4 = new Label();
            label3 = new Label();
            label2 = new Label();
            label1 = new Label();
            button4 = new Button();
            button3 = new Button();
            button2 = new Button();
            btnChooseImage = new Button();
            picAvatar = new PictureBox();
            cboCategory = new ComboBox();
            txtQuantity = new TextBox();
            txtUnitPrice = new TextBox();
            txtProductName = new TextBox();
            txtProductId = new TextBox();
            groupBox1 = new GroupBox();
            statusStrip1 = new StatusStrip();
            lblStatus = new ToolStripStatusLabel();
            dgvProducts = new DataGridView();
            MaSP = new DataGridViewTextBoxColumn();
            TenSP = new DataGridViewTextBoxColumn();
            DanhMuc = new DataGridViewTextBoxColumn();
            DonGia = new DataGridViewTextBoxColumn();
            SoLuong = new DataGridViewTextBoxColumn();
            menuStrip1 = new MenuStrip();
            fileToolStripMenuItem = new ToolStripMenuItem();
            exportCSVToolStripMenuItem = new ToolStripMenuItem();
            exitToolStripMenuItem = new ToolStripMenuItem();
            errorProvider1 = new ErrorProvider(components);
            tableLayoutPanel1.SuspendLayout();
            panel1.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)picAvatar).BeginInit();
            groupBox1.SuspendLayout();
            statusStrip1.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)dgvProducts).BeginInit();
            menuStrip1.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)errorProvider1).BeginInit();
            SuspendLayout();
            // 
            // tableLayoutPanel1
            // 
            tableLayoutPanel1.ColumnCount = 2;
            tableLayoutPanel1.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 35F));
            tableLayoutPanel1.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 65F));
            tableLayoutPanel1.Controls.Add(panel1, 0, 0);
            tableLayoutPanel1.Controls.Add(groupBox1, 1, 0);
            tableLayoutPanel1.Dock = DockStyle.Fill;
            tableLayoutPanel1.Location = new Point(0, 0);
            tableLayoutPanel1.Name = "tableLayoutPanel1";
            tableLayoutPanel1.RowCount = 1;
            tableLayoutPanel1.RowStyles.Add(new RowStyle(SizeType.Percent, 100F));
            tableLayoutPanel1.RowStyles.Add(new RowStyle(SizeType.Absolute, 13F));
            tableLayoutPanel1.RowStyles.Add(new RowStyle(SizeType.Absolute, 27F));
            tableLayoutPanel1.Size = new Size(812, 450);
            tableLayoutPanel1.TabIndex = 0;
            tableLayoutPanel1.Paint += tableLayoutPanel1_Paint;
            // 
            // panel1
            // 
            panel1.Controls.Add(label4);
            panel1.Controls.Add(label3);
            panel1.Controls.Add(label2);
            panel1.Controls.Add(label1);
            panel1.Controls.Add(button4);
            panel1.Controls.Add(button3);
            panel1.Controls.Add(button2);
            panel1.Controls.Add(btnChooseImage);
            panel1.Controls.Add(picAvatar);
            panel1.Controls.Add(cboCategory);
            panel1.Controls.Add(txtQuantity);
            panel1.Controls.Add(txtUnitPrice);
            panel1.Controls.Add(txtProductName);
            panel1.Controls.Add(txtProductId);
            panel1.Dock = DockStyle.Fill;
            panel1.Location = new Point(3, 3);
            panel1.Name = "panel1";
            panel1.Size = new Size(278, 444);
            panel1.TabIndex = 0;
            // 
            // label4
            // 
            label4.AutoSize = true;
            label4.Location = new Point(9, 147);
            label4.Name = "label4";
            label4.Size = new Size(72, 20);
            label4.TabIndex = 7;
            label4.Text = "Số Lượng";
            // 
            // label3
            // 
            label3.AutoSize = true;
            label3.Location = new Point(9, 100);
            label3.Name = "label3";
            label3.Size = new Size(63, 20);
            label3.TabIndex = 7;
            label3.Text = "Đơn Giá";
            label3.Click += label3_Click;
            // 
            // label2
            // 
            label2.AutoSize = true;
            label2.Location = new Point(7, 54);
            label2.Name = "label2";
            label2.Size = new Size(52, 20);
            label2.TabIndex = 7;
            label2.Text = "Tên SP";
            // 
            // label1
            // 
            label1.AutoSize = true;
            label1.Location = new Point(9, 12);
            label1.Name = "label1";
            label1.Size = new Size(50, 20);
            label1.TabIndex = 7;
            label1.Text = "Mã SP";
            label1.Click += label1_Click;
            // 
            // button4
            // 
            button4.Location = new Point(163, 379);
            button4.Name = "button4";
            button4.Size = new Size(94, 29);
            button4.TabIndex = 6;
            button4.Text = "Xóa";
            button4.UseVisualStyleBackColor = true;
            // 
            // button3
            // 
            button3.Location = new Point(26, 379);
            button3.Name = "button3";
            button3.Size = new Size(94, 29);
            button3.TabIndex = 6;
            button3.Text = "Sửa";
            button3.UseVisualStyleBackColor = true;
            // 
            // button2
            // 
            button2.Location = new Point(163, 321);
            button2.Name = "button2";
            button2.Size = new Size(94, 29);
            button2.TabIndex = 6;
            button2.Text = "Thêm";
            button2.UseVisualStyleBackColor = true;
            button2.Click += button2_Click;
            // 
            // btnChooseImage
            // 
            btnChooseImage.Location = new Point(26, 321);
            btnChooseImage.Name = "btnChooseImage";
            btnChooseImage.Size = new Size(94, 29);
            btnChooseImage.TabIndex = 6;
            btnChooseImage.Text = "Chọn ảnh";
            btnChooseImage.UseVisualStyleBackColor = true;
            // 
            // picAvatar
            // 
            picAvatar.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            picAvatar.BorderStyle = BorderStyle.FixedSingle;
            picAvatar.Location = new Point(45, 253);
            picAvatar.Name = "picAvatar";
            picAvatar.Size = new Size(125, 62);
            picAvatar.SizeMode = PictureBoxSizeMode.StretchImage;
            picAvatar.TabIndex = 5;
            picAvatar.TabStop = false;
            // 
            // cboCategory
            // 
            cboCategory.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            cboCategory.FormattingEnabled = true;
            cboCategory.Location = new Point(45, 199);
            cboCategory.Name = "cboCategory";
            cboCategory.Size = new Size(155, 28);
            cboCategory.TabIndex = 4;
            // 
            // txtQuantity
            // 
            txtQuantity.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            txtQuantity.Location = new Point(92, 147);
            txtQuantity.Name = "txtQuantity";
            txtQuantity.Size = new Size(129, 27);
            txtQuantity.TabIndex = 3;
            // 
            // txtUnitPrice
            // 
            txtUnitPrice.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            txtUnitPrice.Location = new Point(92, 100);
            txtUnitPrice.Name = "txtUnitPrice";
            txtUnitPrice.Size = new Size(129, 27);
            txtUnitPrice.TabIndex = 2;
            // 
            // txtProductName
            // 
            txtProductName.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            txtProductName.Location = new Point(92, 51);
            txtProductName.Name = "txtProductName";
            txtProductName.Size = new Size(129, 27);
            txtProductName.TabIndex = 1;
            // 
            // txtProductId
            // 
            txtProductId.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            txtProductId.Location = new Point(92, 5);
            txtProductId.Name = "txtProductId";
            txtProductId.Size = new Size(129, 27);
            txtProductId.TabIndex = 0;
            // 
            // groupBox1
            // 
            groupBox1.Controls.Add(statusStrip1);
            groupBox1.Controls.Add(dgvProducts);
            groupBox1.Controls.Add(menuStrip1);
            groupBox1.Dock = DockStyle.Fill;
            groupBox1.Location = new Point(287, 3);
            groupBox1.Name = "groupBox1";
            groupBox1.Size = new Size(522, 444);
            groupBox1.TabIndex = 1;
            groupBox1.TabStop = false;
            groupBox1.Text = "groupBox1";
            // 
            // statusStrip1
            // 
            statusStrip1.ImageScalingSize = new Size(20, 20);
            statusStrip1.Items.AddRange(new ToolStripItem[] { lblStatus });
            statusStrip1.Location = new Point(3, 415);
            statusStrip1.Name = "statusStrip1";
            statusStrip1.Size = new Size(516, 26);
            statusStrip1.TabIndex = 2;
            statusStrip1.Text = "statusStrip1";
            // 
            // lblStatus
            // 
            lblStatus.Name = "lblStatus";
            lblStatus.Size = new Size(145, 20);
            lblStatus.Text = "Tổng số sản phẩm: 0";
            // 
            // dgvProducts
            // 
            dgvProducts.Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right;
            dgvProducts.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;
            dgvProducts.ColumnHeadersHeightSizeMode = DataGridViewColumnHeadersHeightSizeMode.AutoSize;
            dgvProducts.Columns.AddRange(new DataGridViewColumn[] { MaSP, TenSP, DanhMuc, DonGia, SoLuong });
            dgvProducts.Location = new Point(3, 51);
            dgvProducts.Name = "dgvProducts";
            dgvProducts.RowHeadersWidth = 51;
            dgvProducts.SelectionMode = DataGridViewSelectionMode.FullRowSelect;
            dgvProducts.Size = new Size(516, 390);
            dgvProducts.TabIndex = 0;
            dgvProducts.CellContentClick += dgvProducts_CellContentClick;
            // 
            // MaSP
            // 
            MaSP.DataPropertyName = "Id";
            MaSP.HeaderText = "Mã SP";
            MaSP.MinimumWidth = 6;
            MaSP.Name = "MaSP";
            // 
            // TenSP
            // 
            TenSP.DataPropertyName = "Name";
            TenSP.HeaderText = "Tên SP";
            TenSP.MinimumWidth = 6;
            TenSP.Name = "TenSP";
            // 
            // DanhMuc
            // 
            DanhMuc.DataPropertyName = "CategoryId";
            DanhMuc.HeaderText = "Danh Mục";
            DanhMuc.MinimumWidth = 6;
            DanhMuc.Name = "DanhMuc";
            // 
            // DonGia
            // 
            DonGia.DataPropertyName = "Price";
            dataGridViewCellStyle1.Format = "N0";
            DonGia.DefaultCellStyle = dataGridViewCellStyle1;
            DonGia.HeaderText = "Đơn Giá";
            DonGia.MinimumWidth = 6;
            DonGia.Name = "DonGia";
            // 
            // SoLuong
            // 
            SoLuong.DataPropertyName = "Quantity";
            SoLuong.HeaderText = "Số Lượng";
            SoLuong.MinimumWidth = 6;
            SoLuong.Name = "SoLuong";
            // 
            // menuStrip1
            // 
            menuStrip1.ImageScalingSize = new Size(20, 20);
            menuStrip1.Items.AddRange(new ToolStripItem[] { fileToolStripMenuItem });
            menuStrip1.Location = new Point(3, 23);
            menuStrip1.Name = "menuStrip1";
            menuStrip1.Size = new Size(516, 28);
            menuStrip1.TabIndex = 1;
            menuStrip1.Text = "menuStrip1";
            // 
            // fileToolStripMenuItem
            // 
            fileToolStripMenuItem.DropDownItems.AddRange(new ToolStripItem[] { exportCSVToolStripMenuItem, exitToolStripMenuItem });
            fileToolStripMenuItem.Name = "fileToolStripMenuItem";
            fileToolStripMenuItem.Size = new Size(46, 24);
            fileToolStripMenuItem.Text = "File";
            // 
            // exportCSVToolStripMenuItem
            // 
            exportCSVToolStripMenuItem.Name = "exportCSVToolStripMenuItem";
            exportCSVToolStripMenuItem.ShortcutKeys = Keys.Control | Keys.E;
            exportCSVToolStripMenuItem.Size = new Size(215, 26);
            exportCSVToolStripMenuItem.Text = "Export CSV";
            // 
            // exitToolStripMenuItem
            // 
            exitToolStripMenuItem.Name = "exitToolStripMenuItem";
            exitToolStripMenuItem.ShortcutKeys = Keys.Control | Keys.X;
            exitToolStripMenuItem.Size = new Size(215, 26);
            exitToolStripMenuItem.Text = "Exit";
            // 
            // errorProvider1
            // 
            errorProvider1.ContainerControl = this;
            // 
            // Form1
            // 
            AutoScaleDimensions = new SizeF(8F, 20F);
            AutoScaleMode = AutoScaleMode.Font;
            ClientSize = new Size(812, 450);
            Controls.Add(tableLayoutPanel1);
            Name = "Form1";
            Text = "Form1";
            Load += Form1_Load;
            tableLayoutPanel1.ResumeLayout(false);
            panel1.ResumeLayout(false);
            panel1.PerformLayout();
            ((System.ComponentModel.ISupportInitialize)picAvatar).EndInit();
            groupBox1.ResumeLayout(false);
            groupBox1.PerformLayout();
            statusStrip1.ResumeLayout(false);
            statusStrip1.PerformLayout();
            ((System.ComponentModel.ISupportInitialize)dgvProducts).EndInit();
            menuStrip1.ResumeLayout(false);
            menuStrip1.PerformLayout();
            ((System.ComponentModel.ISupportInitialize)errorProvider1).EndInit();
            ResumeLayout(false);
        }

        #endregion

        private TableLayoutPanel tableLayoutPanel1;
        private Panel panel1;
        private TextBox txtQuantity;
        private TextBox txtUnitPrice;
        private TextBox txtProductName;
        private TextBox txtProductId;
        private PictureBox picAvatar;
        private ComboBox cboCategory;
        private Button button4;
        private Button button3;
        private Button button2;
        private Button btnChooseImage;
        private ErrorProvider errorProvider1;
        private GroupBox groupBox1;
        private DataGridView dgvProducts;
        private MenuStrip menuStrip1;
        private StatusStrip statusStrip1;
        private ToolStripMenuItem fileToolStripMenuItem;
        private ToolStripMenuItem exportCSVToolStripMenuItem;
        private ToolStripMenuItem exitToolStripMenuItem;
        private ToolStripStatusLabel lblStatus;
        private Label label4;
        private Label label3;
        private Label label2;
        private Label label1;
        private DataGridViewTextBoxColumn MaSP;
        private DataGridViewTextBoxColumn TenSP;
        private DataGridViewTextBoxColumn DanhMuc;
        private DataGridViewTextBoxColumn DonGia;
        private DataGridViewTextBoxColumn SoLuong;
    }
}
<img width="777" height="424" alt="image" src="https://github.com/user-attachments/assets/90f6a7fc-782d-47ef-bb8a-c07871ee895f" />
