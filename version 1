import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { Toaster, toast } from 'react-hot-toast';

const PAGE_SIZE_OPTIONS = [10, 20, 40, 80, 100];

function getStatusColor(quantity) {
  if (quantity === 0) return 'bg-red-100 text-red-700';
  if (quantity <= 10) return 'bg-yellow-100 text-yellow-700';
  return 'bg-blue-100 text-blue-700';
}

export default function InventoryManagement() {
  const [products, setProducts] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterStatus, setFilterStatus] = useState('all');
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editingProduct, setEditingProduct] = useState(null);
  const [formData, setFormData] = useState({ name: '', description: '', quantity: 0 });
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // Bulk actions and activity log
  const [selectedProducts, setSelectedProducts] = useState([]);
  const [activityLog, setActivityLog] = useState([]);

  // Pagination
  const [pageSize, setPageSize] = useState(PAGE_SIZE_OPTIONS[0]);
  const [currentPage, setCurrentPage] = useState(1);

  useEffect(() => {
    fetchProducts();
  }, []);

  const fetchProducts = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await axios.get('/api/products');
      const data = response.data;
      if (Array.isArray(data)) {
        setProducts(data);
      } else if (data && Array.isArray(data.products)) {
        setProducts(data.products);
      } else {
        setProducts([]);
      }
    } catch (e) {
      setError('❌ Unable to load products. Please check your API/server.');
      setProducts([]);
    }
    setLoading(false);
  };

  const filteredProducts = products.filter((product) => {
    const matchesSearch =
      product.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      product.description.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesStatus =
      filterStatus === 'all'
        ? true
        : filterStatus === 'low'
        ? product.quantity <= 10 && product.quantity > 0
        : filterStatus === 'out'
        ? product.quantity === 0
        : product.quantity > 10;
    return matchesSearch && matchesStatus;
  });

  const totalRecords = filteredProducts.length;
  const totalPages = Math.max(1, Math.ceil(totalRecords / pageSize));
  const paginatedProducts = filteredProducts.slice(
    (currentPage - 1) * pageSize,
    currentPage * pageSize
  );

  // Bulk select helpers
  const allSelected = paginatedProducts.length > 0 && paginatedProducts.every(p => selectedProducts.includes(p.id));
  const anySelected = selectedProducts.length > 0;

  // Activity log helper
  const logActivity = (action, product) => {
    setActivityLog(log => [
      { action, product: product.name || product.id, time: new Date().toLocaleString(), user: 'Admin' },
      ...log.slice(0, 19)
    ]);
  };

  // Handlers
  const handleAddClick = () => {
    setEditingProduct(null);
    setFormData({ name: '', description: '', quantity: 0 });
    setIsModalOpen(true);
  };

  const handleEditClick = (product) => {
    setEditingProduct(product);
    setFormData({ name: product.name, description: product.description, quantity: product.quantity });
    setIsModalOpen(true);
  };

  const handleDeleteClick = async (id) => {
    if (window.confirm('Are you sure you want to delete this product? This action cannot be undone.')) {
      try {
        await axios.delete(`/api/products/${id}`);
        logActivity('Deleted', products.find(p => p.id === id) || { name: id });
        fetchProducts();
      } catch {
        setError('❌ Failed to delete product. Try again!');
      }
    }
  };

  // Bulk actions
  const handleBulkDelete = async () => {
    if (window.confirm('Delete selected products?')) {
      await Promise.all(selectedProducts.map(id => axios.delete(`/api/products/${id}`)));
      setSelectedProducts([]);
      selectedProducts.forEach(id => logActivity('Deleted', products.find(p => p.id === id) || { name: id }));
      fetchProducts();
    }
  };

  const handleBulkOutOfStock = async () => {
    await Promise.all(selectedProducts.map(id =>
      axios.patch(`/api/products/${id}`, { quantity: 0 })
    ));
    setSelectedProducts([]);
    selectedProducts.forEach(id => logActivity('Marked Out of Stock', products.find(p => p.id === id) || { name: id }));
    fetchProducts();
  };

  // Import only (no export)
  const importCSV = (e) => {
    const file = e.target.files[0];
    if (file) {
      Papa.parse(file, {
        header: true,
        complete: async (results) => {
          for (const row of results.data) {
            if (row.name && row.quantity) {
              await axios.post('/api/products', row);
              logActivity('Imported', row);
            }
          }
          fetchProducts();
        }
      });
    }
  };

  return (
    <div className="w-screen h-screen bg-blue-50 flex flex-col p-2 sm:p-4 md:p-8 overflow-auto">
      <Toaster position="top-center" />
      <h1 className="text-3xl text-blue-900 font-extrabold tracking-tight drop-shadow mb-6">Inventory Management</h1>

      {/* Controls Row: Search, Status+Add, Records, Total */}
      <div className="flex flex-col md:flex-row md:items-center md:space-x-4 mb-4 w-full">
        {/* Search */}
        <div className="flex flex-col md:flex-row flex-grow gap-4 items-center">
          <div className="relative w-full md:w-96">
            <span className="absolute left-3 top-1/2 transform -translate-y-1/2 text-blue-300 pointer-events-none">
              <svg width="20" height="20" fill="none" stroke="currentColor" strokeWidth="2" viewBox="0 0 20 20">
                <circle cx="9" cy="9" r="7"/>
                <path d="M16 16l-3.5-3.5"/>
              </svg>
            </span>
            <input
              type="text"
              placeholder="Search products..."
              className="border border-blue-200 bg-white text-blue-900 placeholder-blue-400 px-4 pl-10 py-2 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-300 focus:border-transparent transition duration-300 w-full"
              value={searchTerm}
              onChange={(e) => {
                setSearchTerm(e.target.value);
                setCurrentPage(1);
              }}
            />
          </div>
          {/* Status dropdown and Add Product button */}
          <div className="flex items-center space-x-2 w-full md:w-auto">
            <select
              className="border border-blue-200 bg-white text-blue-900 px-4 py-2 pr-8 border-r-8 border-r-transparent rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-300 transition"
              value={filterStatus}
              onChange={(e) => {
                setFilterStatus(e.target.value);
                setCurrentPage(1);
              }}
            >
              <option value="all">All Statuses</option>
              <option value="low">Low Stock</option>
              <option value="out">Out of Stock</option>
              <option value="high">In Stock</option>
            </select>
            <button
              onClick={handleAddClick}
              className="bg-blue-100 hover:bg-blue-200 text-blue-700 rounded-lg shadow px-5 py-2 font-semibold border border-blue-200 transition"
            >
              + Add Product
            </button>
          </div>
        </div>
        {/* Records, Total */}
        <div className="flex items-center space-x-2 mt-4 md:mt-0">
          <span className="font-semibold text-blue-900">Records:</span>
          {PAGE_SIZE_OPTIONS.map((size) => (
            <button
              key={size}
              className={`px-2 py-1 rounded border font-semibold ${
                pageSize === size
                  ? 'bg-blue-100 text-blue-700 border-blue-200'
                  : 'bg-white text-blue-700 border-blue-200'
              } hover:bg-blue-50 transition`}
              onClick={() => setPageSize(size)}
            >
              {size}
            </button>
          ))}
          <span className="ml-2 text-blue-600 font-medium">Total: {totalRecords}</span>
        </div>
      </div>

      {/* Bulk Actions and Import */}
      <div className="flex gap-2 mb-4">
        <label className="bg-blue-100 hover:bg-blue-200 text-blue-700 px-3 py-1 rounded border border-blue-200 cursor-pointer">
          Import CSV
          <input type="file" accept=".csv" onChange={importCSV} className="hidden" />
        </label>
        <button
          onClick={handleBulkDelete}
          disabled={!anySelected}
          className="bg-red-100 hover:bg-red-200 text-red-700 px-3 py-1 rounded border border-red-200 disabled:opacity-50"
        >
          Delete Selected
        </button>
        <button
          onClick={handleBulkOutOfStock}
          disabled={!anySelected}
          className="bg-yellow-100 hover:bg-yellow-200 text-yellow-700 px-3 py-1 rounded border border-yellow-200 disabled:opacity-50"
        >
          Mark Out of Stock
        </button>
      </div>

      {/* Products Table */}
      <div className="overflow-x-auto rounded-lg border border-blue-100 bg-white flex-1 shadow-sm">
        <table className="min-w-full divide-y divide-blue-100 text-sm">
          <thead className="bg-blue-50 text-blue-900 font-semibold">
            <tr>
              <th className="px-2 py-3">
                <input
                  type="checkbox"
                  checked={allSelected}
                  onChange={e => {
                    if (e.target.checked) {
                      setSelectedProducts(paginatedProducts.map(p => p.id));
                    } else {
                      setSelectedProducts([]);
                    }
                  }}
                />
              </th>
              <th className="px-4 py-3 text-left uppercase">Product</th>
              <th className="px-4 py-3 text-left uppercase">Description</th>
              <th className="px-4 py-3 text-left uppercase">Quantity</th>
              <th className="px-4 py-3 text-left uppercase">Status</th>
              <th className="px-4 py-3 text-left uppercase">Actions</th>
            </tr>
          </thead>
          <tbody>
            {loading ? (
              <tr>
                <td colSpan={6} className="text-center p-6 text-blue-400">
                  Loading...
                </td>
              </tr>
            ) : paginatedProducts.length === 0 ? (
              <tr>
                <td colSpan={6} className="text-center p-6 text-blue-400">
                  No products found.
                </td>
              </tr>
            ) : (
              paginatedProducts.map((product, idx) => (
                <tr key={product.id} className={idx % 2 === 0 ? 'bg-white' : 'bg-blue-50'}>
                  <td className="px-2 py-3">
                    <input
                      type="checkbox"
                      checked={selectedProducts.includes(product.id)}
                      onChange={e => {
                        if (e.target.checked) {
                          setSelectedProducts([...selectedProducts, product.id]);
                        } else {
                          setSelectedProducts(selectedProducts.filter(id => id !== product.id));
                        }
                      }}
                    />
                  </td>
                  <td className="px-4 py-3">{product.name}</td>
                  <td className="px-4 py-3">{product.description}</td>
                  <td className="px-4 py-3">{product.quantity}</td>
                  <td className="px-4 py-3">
                    <span className={`px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusColor(product.quantity)}`}>
                      {product.quantity === 0
                        ? 'Out of Stock'
                        : product.quantity <= 10
                        ? 'Low Stock'
                        : 'In Stock'}
                    </span>
                  </td>
                  <td className="px-4 py-3">
                    <button
                      onClick={() => handleEditClick(product)}
                      className="text-blue-500 hover:text-blue-700 mr-4 font-semibold"
                    >
                      Edit
                    </button>
                    <button
                      onClick={() => handleDeleteClick(product.id)}
                      className="text-red-400 hover:text-red-600 font-semibold"
                    >
                      Delete
                    </button>
                  </td>
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>

      {/* Pagination Controls */}
      <div className="flex flex-wrap justify-center items-center gap-2 mt-4">
        {Array.from({ length: totalPages }, (_, i) => i + 1).map((page) => (
          <button
            key={page}
            className={`px-3 py-1 rounded border font-semibold ${
              currentPage === page
                ? 'bg-blue-100 text-blue-700 border-blue-200'
                : 'bg-white text-blue-700 border-blue-200'
            } hover:bg-blue-50 transition`}
            onClick={() => setCurrentPage(page)}
            aria-current={currentPage === page ? "page" : undefined}
          >
            {page}
          </button>
        ))}
      </div>

      {/* Activity Log */}
      <div className="mt-6 bg-white rounded shadow p-4">
        <h3 className="text-lg font-semibold text-blue-900 mb-2">Recent Activity</h3>
        <div className="space-y-2 text-sm">
          {activityLog.map((entry, index) => (
            <div key={index} className="flex items-center text-blue-700">
              <span className="text-xs text-blue-400 mr-2">{entry.time}</span>
              <span className="font-medium">{entry.user}</span>
              <span className="mx-1">{entry.action}</span>
              <span className="text-blue-900 font-semibold">{entry.product}</span>
            </div>
          ))}
        </div>
      </div>

      {/* Modal */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-black bg-opacity-30 flex items-center justify-center transition-opacity duration-300 z-50">
          <div className="bg-white rounded-lg shadow-lg p-6 max-w-md w-full border border-blue-100 transform transition-transform duration-300 scale-100">
            <h2 className="text-blue-900 font-bold text-2xl mb-4 flex items-center gap-2">
              {editingProduct ? (
                <>
                  <span className="inline-block w-6 h-6 bg-blue-100 text-blue-700 rounded-full flex items-center justify-center">✏️</span>
                  Edit Product
                </>
              ) : (
                <>
                  <span className="inline-block w-6 h-6 bg-blue-100 text-blue-700 rounded-full flex items-center justify-center">➕</span>
                  Add New Product
                </>
              )}
            </h2>
            <form
              onSubmit={async (e) => {
                e.preventDefault();
                if (editingProduct) {
                  try {
                    await axios.put(`/api/products/${editingProduct.id}`, formData);
                    toast.success(`${formData.name} has been edited to the inventory`);
                    logActivity('Edited', formData);
                  } catch {
                    setError('❌ Failed to save product. Please check your input or try again.');
                  }
                } else {
                  try {
                    await axios.post('/api/products', formData);
                    toast.success(`${formData.name} has been added to the inventory`);
                    logActivity('Added', formData);
                  } catch {
                    setError('❌ Failed to save product. Please check your input or try again.');
                  }
                }
                setIsModalOpen(false);
                fetchProducts();
              }}
            >
              <div className="mb-4">
                <label className="block text-sm font-medium mb-1 text-blue-900">Product Name</label>
                <input
                  type="text"
                  name="name"
                  value={formData.name}
                  onChange={(e) =>
                    setFormData({
                      ...formData,
                      [e.target.name]: e.target.name === 'quantity' ? Number(e.target.value) : e.target.value,
                    })
                  }
                  className="w-full border border-blue-200 p-2 rounded focus:ring-2 focus:ring-blue-300 outline-none transition"
                  required
                  placeholder="Enter product name"
                />
              </div>
              <div className="mb-4">
                <label className="block text-sm font-medium mb-1 text-blue-900">Description</label>
                <textarea
                  name="description"
                  value={formData.description}
                  onChange={(e) =>
                    setFormData({ ...formData, [e.target.name]: e.target.value })
                  }
                  className="w-full border border-blue-200 p-2 rounded focus:ring-2 focus:ring-blue-300 outline-none transition"
                  rows={3}
                  placeholder="Enter product description"
                />
              </div>
              <div className="mb-4">
                <label className="block text-sm font-medium mb-1 text-blue-900">Quantity</label>
                <input
                  type="number"
                  name="quantity"
                  value={formData.quantity}
                  onChange={(e) =>
                    setFormData({ ...formData, [e.target.name]: Number(e.target.value) })
                  }
                  className="w-full border border-blue-200 p-2 rounded focus:ring-2 focus:ring-blue-300 outline-none transition"
                  min={0}
                  required
                  placeholder="Enter quantity"
                />
              </div>
              <div className="flex justify-end gap-2 mt-4">
                <button
                  type="button"
                  onClick={() => setIsModalOpen(false)}
                  className="border border-gray-200 hover:bg-gray-100 rounded px-4 py-2 transition"
                >
                  Cancel
                </button>
                <button
                  type="submit"
                  className="bg-blue-100 hover:bg-blue-200 text-blue-700 rounded px-4 py-2 transition font-semibold border border-blue-200"
                >
                  {editingProduct ? 'Update' : 'Add'} Product
                </button>
              </div>
            </form>
            <div className="mt-6 text-center text-blue-300 text-xs">
              {editingProduct
                ? 'Update the product details and click Update Product.'
                : 'Fill in the details and click Add Product to create a new entry.'}
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
