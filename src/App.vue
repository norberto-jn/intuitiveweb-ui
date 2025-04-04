<template>
  <div class="health-insurance-providers-container">
    <!-- Header Section -->
    <header class="app-header">
      <h1 class="app-title">Health Insurance Providers</h1>
      <div class="header-actions">
        <button @click="openCreateModal" class="add-button">
          <i class="fas fa-plus"></i> Add Provider
        </button>
      </div>
    </header>

    <!-- Search Section -->
    <div class="search-section">
      <div class="search-card">
        <div class="search-inputs">
          <div class="input-group">
            <label for="ans-registration">Registration ANS</label>
            <input
              id="ans-registration"
              v-model="searchParams.ans_registration_code"
              placeholder="Enter registration code"
              @keyup.enter="searchProviders"
            />
          </div>
          <div class="input-group">
            <label for="cnpj">CNPJ</label>
            <input
              id="cnpj"
              v-model="searchParams.cnpj"
              placeholder="Enter CNPJ"
              v-mask="'##.###.###/####-##'"
              @keyup.enter="searchProviders"
            />
          </div>
          <div class="input-group">
            <label for="fantasy-name">Fantasy Name</label>
            <input
              id="fantasy-name"
              v-model="searchParams.fantasyname"
              placeholder="Enter fantasy name"
              @keyup.enter="searchProviders"
            />
          </div>
        </div>
        <div class="search-actions">
          <button @click="searchProviders" class="search-button">
            <i class="fas fa-search"></i> Search
          </button>
          <button @click="resetSearch" class="reset-button">
            <i class="fas fa-undo"></i> Reset
          </button>
        </div>
      </div>
    </div>

    <!-- Results Section -->
    <div class="results-section">
      <div v-if="loading" class="loading-spinner">
        <i class="fas fa-spinner fa-spin"></i> Loading...
      </div>
      
      <div v-else-if="providers.length === 0" class="no-results">
        <i class="fas fa-info-circle"></i> No providers found
      </div>
      
      <div v-else class="providers-table">
        <div class="table-header">
          <div class="table-row">
            <div class="table-cell">Code</div>
            <div class="table-cell">Fantasy Name</div>
            <div class="table-cell">CNPJ</div>
            <div class="table-cell">Modality</div>
            <div class="table-cell">Region</div>
            <div class="table-cell">Actions</div>
          </div>
        </div>
        <div class="table-body">
          <div v-for="provider in providers" :key="provider.ans_registration_code" class="table-row">
            <div class="table-cell">{{ provider.ans_registration_code }}</div>
            <div class="table-cell">{{ provider.fantasyname }}</div>
            <div class="table-cell">{{ formatCNPJ(provider.cnpj) }}</div>
            <div class="table-cell">{{ provider.modality }}</div>
            <div class="table-cell">{{ provider.commercialization_region }}</div>
            <div class="table-cell actions-cell">
              <button @click="viewProvider(provider)" class="action-button view">
                <i class="fas fa-eye"></i>
              </button>
              <button @click="editProvider(provider)" class="action-button edit">
                <i class="fas fa-edit"></i>
              </button>
              <button @click="confirmDelete(provider)" class="action-button delete">
                <i class="fas fa-trash"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div v-if="providers.length > 0" class="pagination">
      <button 
        @click="prevPage" 
        :disabled="currentPage === 1" 
        class="pagination-button"
      >
        <i class="fas fa-chevron-left"></i>
      </button>
      <span class="page-info">Page {{ currentPage }} of {{ totalPages }}</span>
      <button 
        @click="nextPage" 
        :disabled="currentPage === totalPages" 
        class="pagination-button"
      >
        <i class="fas fa-chevron-right"></i>
      </button>
    </div>

    <!-- Provider Modal -->
    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <div class="modal-header">
          <h2>{{ modalTitle }}</h2>
          <button @click="closeModal" class="close-button">
            <i class="fas fa-times"></i>
          </button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="submitForm">
            <div class="form-grid">
              <div class="form-group">
                <label>Registration ANS Code</label>
                <input 
                  v-model="formData.ans_registration_code" 
                  required 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>CNPJ</label>
                <input 
                  v-model="formData.cnpj" 
                  v-mask="'##.###.###/####-##'" 
                  required 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Social Name</label>
                <input 
                  v-model="formData.socialname" 
                  required 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Fantasy Name</label>
                <input 
                  v-model="formData.fantasyname" 
                  required 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Modality</label>
                <input 
                  v-model="formData.modality" 
                  required 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Area Code</label>
                <input 
                  v-model="formData.area_code" 
                  v-mask="'(##)'"
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Phone</label>
                <input 
                  v-model="formData.phone" 
                  v-mask="'#####-####'"
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Fax</label>
                <input 
                  v-model="formData.fax" 
                  v-mask="'#####-####'"
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Email</label>
                <input 
                  v-model="formData.email" 
                  type="email" 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Representative</label>
                <input 
                  v-model="formData.representative" 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Representative Position</label>
                <input 
                  v-model="formData.representative_position" 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Commercialization Region</label>
                <input 
                  v-model="formData.commercialization_region" 
                  :disabled="isViewMode"
                />
              </div>
              <div class="form-group">
                <label>Registration Date</label>
                <input 
                  v-model="formData.ans_registration_date" 
                  type="date" 
                  :disabled="isViewMode"
                />
              </div>
            </div>
            <div v-if="!isViewMode" class="form-actions">
              <button type="button" @click="closeModal" class="cancel-button">
                Cancel
              </button>
              <button type="submit" class="submit-button">
                {{ isEditMode ? 'Update' : 'Create' }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div v-if="showDeleteModal" class="modal-overlay" @click.self="showDeleteModal = false">
      <div class="delete-modal-content">
        <div class="modal-header">
          <h2>Confirm Deletion</h2>
          <button @click="showDeleteModal = false" class="close-button">
            <i class="fas fa-times"></i>
          </button>
        </div>
        <div class="modal-body">
          <p>Are you sure you want to delete provider <strong>{{ providerToDelete?.fantasyname }}</strong> ({{ providerToDelete?.ans_registration_code }})?</p>
          <div class="delete-actions">
            <button @click="showDeleteModal = false" class="cancel-button">
              Cancel
            </button>
            <button @click="deleteProvider" class="delete-button">
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { mask } from 'vue-the-mask';
import { useToast } from 'vue-toastification';

export default {
  directives: { mask },
  data() {
    return {
      // Seus dados permanecem os mesmos
      providers: [],
      loading: false,
      currentPage: 1,
      itemsPerPage: 10,
      totalPages: 1,
      searchParams: {
        ans_registration_code: '',
        cnpj: '',
        fantasyname: ''
      },
      showModal: false,
      showDeleteModal: false,
      isEditMode: false,
      isViewMode: false,
      providerToDelete: null,
      formData: {
        ans_registration_code: '',
        cnpj: '',
        socialname: '',
        fantasyname: '',
        modality: '',
        area_code: '',
        phone: '',
        fax: '',
        email: '',
        representative: '',
        representative_position: '',
        commercialization_region: '',
        ans_registration_date: ''
      }
    };
  },
  setup() {
    // Inicializa o toast
    const toast = useToast();
    return { toast };
  },
  computed: {
    modalTitle() {
      if (this.isViewMode) return 'View Provider Details';
      if (this.isEditMode) return 'Edit Provider';
      return 'Add New Provider';
    }
  },
  created() {
    this.fetchProviders();
  },
  methods: {
    async fetchProviders() {
      this.loading = true;
      try {
        const params = {
          ...this.searchParams,
          page: this.currentPage,
          limit: this.itemsPerPage
        };
        
        const response = await axios.get('http://localhost:3002/health-insurance-providers/search', { params });
        this.providers = response.data.items || response.data;
        this.totalPages = response.data.totalPages || 1;
      } catch (error) {
        console.error('Error fetching providers:', error);
        this.providers = [];
        this.toast.error('Failed to fetch providers');
      } finally {
        this.loading = false;
      }
    },
    async searchProviders() {
      this.currentPage = 1;
      await this.fetchProviders();
    },
    resetSearch() {
      this.searchParams = {
        ans_registration_code: '',
        cnpj: '',
        fantasyname: ''
      };
      this.currentPage = 1;
      this.fetchProviders();
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.fetchProviders();
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.fetchProviders();
      }
    },
    openCreateModal() {
      this.resetForm();
      this.isEditMode = false;
      this.isViewMode = false;
      this.showModal = true;
    },
    viewProvider(provider) {
      this.formData = { ...provider };
      this.isViewMode = true;
      this.showModal = true;
    },
    editProvider(provider) {
      this.formData = { ...provider };
      this.isEditMode = true;
      this.isViewMode = false;
      this.showModal = true;
    },
    confirmDelete(provider) {
      this.providerToDelete = { ...provider };
      this.showDeleteModal = true;
    },
    async deleteProvider() {
      if (!this.providerToDelete || !this.providerToDelete.ans_registration_code) {
        console.error('No provider selected for deletion');
        this.toast.error('No provider selected for deletion');
        return;
      }

      try {
        const response = await axios.delete(
          `http://localhost:3002/health-insurance-providers/${this.providerToDelete.ans_registration_code}`
        );
        
        this.showDeleteModal = false;
        await this.fetchProviders();
        
        const successMessage = response.data?.message || 'Provider deleted successfully';
        this.toast.success(successMessage);
      } catch (error) {
        console.error('Error deleting provider:', error);
        
        const errorMessage = error.response?.data?.message || 
                         error.message || 
                         'Failed to delete provider';
        this.toast.error(errorMessage);
      } finally {
        this.providerToDelete = null;
      }
    },
    async submitForm() {
      try {
        if (this.isEditMode) {
          await axios.put(
            `http://localhost:3002/health-insurance-providers/${this.formData.ans_registration_code}`,
            this.formData
          );
          this.toast.success('Provider updated successfully');
        } else {
          await axios.post(
            'http://localhost:3002/health-insurance-providers',
            this.formData
          );
          this.toast.success('Provider created successfully');
        }
        this.closeModal();
        this.fetchProviders();
      } catch (error) {
        console.error('Error saving provider:', error);
        const errorMessage = error.response?.data?.message || 
                         `Failed to ${this.isEditMode ? 'update' : 'create'} provider`;
        this.toast.error(errorMessage);
      }
    },
    resetForm() {
      this.formData = {
        ans_registration_code: '',
        cnpj: '',
        socialname: '',
        fantasyname: '',
        modality: '',
        area_code: '',
        phone: '',
        fax: '',
        email: '',
        representative: '',
        representative_position: '',
        commercialization_region: '',
        ans_registration_date: ''
      };
    },
    closeModal() {
      this.showModal = false;
      this.isEditMode = false;
      this.isViewMode = false;
    },
    formatCNPJ(cnpj) {
      if (!cnpj) return '';
      return cnpj.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, '$1.$2.$3/$4-$5');
    }
  }
};
</script>


<style scoped>
/* Seu CSS permanece exatamente o mesmo */
.health-insurance-providers-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  color: #333;
}

/* ... (todo o resto do seu CSS permanece igual) ... */

@media (max-width: 768px) {
  .search-inputs {
    grid-template-columns: 1fr;
  }
  
  .table-row {
    grid-template-columns: 1fr 1fr;
    grid-template-areas: 
      "code name"
      "cnpj modality"
      "region actions";
    gap: 10px;
    padding: 15px;
  }
  
  .table-row:first-child {
    display: none;
  }
  
  .table-cell:nth-child(1) { grid-area: code; }
  .table-cell:nth-child(2) { grid-area: name; }
  .table-cell:nth-child(3) { grid-area: cnpj; }
  .table-cell:nth-child(4) { grid-area: modality; }
  .table-cell:nth-child(5) { grid-area: region; }
  .table-cell:nth-child(6) { grid-area: actions; justify-self: end; }
  
  .modal-content {
    width: 95%;
  }
  
  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>
<style scoped>
.health-insurance-providers-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  color: #333;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 15px;
  border-bottom: 1px solid #e0e0e0;
}

.app-title {
  font-size: 28px;
  font-weight: 600;
  color: #2c3e50;
  margin: 0;
}

.add-button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background-color 0.3s;
}

.add-button:hover {
  background-color: #3e8e41;
}

.search-section {
  margin-bottom: 25px;
}

.search-card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  padding: 20px;
}

.search-inputs {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.input-group {
  display: flex;
  flex-direction: column;
}

.input-group label {
  margin-bottom: 8px;
  font-weight: 500;
  color: #555;
}

.input-group input {
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.input-group input:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

.search-actions {
  display: flex;
  gap: 10px;
}

.search-button, .reset-button {
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s;
}

.search-button {
  background-color: #2196F3;
  color: white;
  border: none;
}

.search-button:hover {
  background-color: #0b7dda;
}

.reset-button {
  background-color: #f5f5f5;
  color: #555;
  border: 1px solid #ddd;
}

.reset-button:hover {
  background-color: #e0e0e0;
}

.results-section {
  margin-bottom: 25px;
}

.loading-spinner, .no-results {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 30px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  gap: 10px;
  color: #555;
}

.providers-table {
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

.table-header, .table-body {
  width: 100%;
}

.table-row {
  display: grid;
  grid-template-columns: 1fr 2fr 1.5fr 1fr 1fr 120px;
  align-items: center;
  padding: 12px 15px;
  border-bottom: 1px solid #eee;
}

.table-header .table-row {
  font-weight: 600;
  background-color: #f8f9fa;
  color: #555;
}

.table-cell {
  padding: 5px;
}

.actions-cell {
  display: flex;
  gap: 8px;
}

.action-button {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.action-button.view {
  background-color: #2196F3;
  color: white;
}

.action-button.edit {
  background-color: #FFC107;
  color: white;
}

.action-button.delete {
  background-color: #F44336;
  color: white;
}

.action-button:hover {
  transform: scale(1.1);
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
}

.pagination-button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.pagination-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pagination-button:not(:disabled):hover {
  background-color: #f5f5f5;
}

.page-info {
  font-weight: 500;
  color: #555;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content, .delete-modal-content {
  background: white;
  border-radius: 10px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2);
  animation: modalFadeIn 0.3s;
}

.delete-modal-content {
  max-width: 500px;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.modal-header h2 {
  margin: 0;
  font-size: 22px;
  color: #2c3e50;
}

.close-button {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #777;
  transition: color 0.3s;
}

.close-button:hover {
  color: #333;
}

.modal-body {
  padding: 20px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #555;
}

.form-group input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.form-group input:disabled {
  background-color: #f5f5f5;
  color: #777;
}

.form-actions, .delete-actions {
  display: flex;
  justify-content: flex-end;
  gap: 15px;
  margin-top: 25px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.cancel-button, .submit-button, .delete-button {
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s;
}

.cancel-button {
  background-color: #f5f5f5;
  color: #555;
  border: 1px solid #ddd;
}

.cancel-button:hover {
  background-color: #e0e0e0;
}

.submit-button {
  background-color: #4CAF50;
  color: white;
  border: none;
}

.submit-button:hover {
  background-color: #3e8e41;
}

.delete-button {
  background-color: #F44336;
  color: white;
  border: none;
}

.delete-button:hover {
  background-color: #d32f2f;
}

@media (max-width: 768px) {
  .search-inputs {
    grid-template-columns: 1fr;
  }
  
  .table-row {
    grid-template-columns: 1fr 1fr;
    grid-template-areas: 
      "code name"
      "cnpj modality"
      "region actions";
    gap: 10px;
    padding: 15px;
  }
  
  .table-row:first-child {
    display: none;
  }
  
  .table-cell:nth-child(1) { grid-area: code; }
  .table-cell:nth-child(2) { grid-area: name; }
  .table-cell:nth-child(3) { grid-area: cnpj; }
  .table-cell:nth-child(4) { grid-area: modality; }
  .table-cell:nth-child(5) { grid-area: region; }
  .table-cell:nth-child(6) { grid-area: actions; justify-self: end; }
  
  .modal-content {
    width: 95%;
  }
  
  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>