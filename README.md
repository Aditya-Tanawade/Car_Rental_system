
Assign Project.html
<!-- Confirmation Modal -->
<div *ngIf="showConfirmModal" class="modal-overlay">
  <div class="modal-container confirmation-modal">
    <div class="modal-header">
      <h2>Confirm Project Assignment</h2>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to assign this project?</p>
      <div class="confirmation-details">
        <div class="detail-row">
          <span class="detail-label">Project:</span>
          <span class="detail-value">{{ confirmationData?.projectName }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Team Lead:</span>
          <span class="detail-value">{{ confirmationData?.teamLeadId }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Budget:</span>
          <span class="detail-value">₹{{ confirmationData?.budgetMin }}L - ₹{{ confirmationData?.budgetMax }}L</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Duration:</span>
          <span class="detail-value">{{ confirmationData?.startDate }} to {{ confirmationData?.endDate }}</span>
        </div>
      </div>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="cancelConfirmation()">Cancel</button>
      <button type="button" class="btn btn-primary" (click)="confirmAssignment()">Confirm</button>
    </div>
  </div>
</div>

<!-- Success Modal -->
<div *ngIf="showSuccessModal" class="modal-overlay">
  <div class="modal-container success-modal">
    <div class="modal-header success-header">
      <div class="success-icon">✓</div>
      <h2>Success!</h2>
    </div>
    <div class="modal-body">
      <p>{{ successMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeSuccessModal()">OK</button>
    </div>
  </div>
</div>

<!-- Error Modal -->
<div *ngIf="showErrorModal" class="modal-overlay">
  <div class="modal-container error-modal">
    <div class="modal-header error-header">
      <div class="error-icon">✕</div>
      <h2>Error</h2>
    </div>
    <div class="modal-body">
      <p>{{ errorMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeErrorModal()">OK</button>
    </div>
  </div>
</div>





Assign-project.ts
import { Component, OnInit } from '@angular/core';
import { PmService } from '../../service/pm-service';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { ProjectResponseDTO } from '../../dto/ProjectResponseDTO';
import { ProjectRequestDTO } from '../../dto/ProjectRequestDTO';

@Component({
  selector: 'app-assign-project',
  imports: [FormsModule, CommonModule],
  templateUrl: './assign-project.html',
  styleUrl: './assign-project.css'
})
export class AssignProject implements OnInit {

  showModal = false;
  showConfirmModal = false;
  showSuccessModal = false;
  showErrorModal = false;
  
  errorMessage: string = '';
  successMessage: string = '';
  confirmationData: any = null;
  
  teamleadIds: string[] = [];
  assignedProject: ProjectResponseDTO | null = null;
  loginPmId: string = "EMP1002";
  isLoading: boolean = true;
  
  projectRequest: ProjectRequestDTO = new ProjectRequestDTO();

  constructor(private pmService: PmService) {}

  ngOnInit(): void {
    this.checkAssignedProject();
  }

  checkAssignedProject(): void {
    this.isLoading = true;
    this.pmService.getAssignedProjects(this.loginPmId).subscribe({
      next: (data) => {
        console.log('Assigned project:', data);
        if (data && data.projectId) {
          this.assignedProject = data;
        } else {
          this.assignedProject = null;
        }
        this.isLoading = false;
      },
      error: (error) => {
        console.error('Error fetching assigned project:', error);
        this.assignedProject = null;
        this.isLoading = false;
      }
    });
  }

  fetchTeamLeadIds(): void {
    this.pmService.findTeamLeadIDs().subscribe({
      next: (ids: string[]) => {
        this.teamleadIds = ids;
        console.log('Team Lead IDs fetched:', ids);
      },
      error: (error) => {
        console.error('Error fetching team lead IDs:', error);
        this.showError('Failed to fetch team lead IDs. Please try again.');
      }
    });
  }

  openModal(): void {
    this.fetchTeamLeadIds();
    this.showModal = true;
  }

  closeModal(): void {
    this.showModal = false;
    this.resetForm();
  }

  resetForm(): void {
    this.projectRequest = new ProjectRequestDTO();
    this.projectRequest.status = 'READY_FOR_ASSIGNMENT';
    this.projectRequest.budgetBandMin = 20;
    this.projectRequest.budgetBandMax = 100;
  }

  onSubmit(event: Event): void {
    event.preventDefault();
    
    if (!this.validateForm()) {
      return;
    }

    // Show confirmation modal
    this.confirmationData = {
      projectName: this.projectRequest.name,
      teamLeadId: this.projectRequest.teamLeadId,
      budgetMin: this.projectRequest.budgetBandMin,
      budgetMax: this.projectRequest.budgetBandMax,
      startDate: this.formatDate(this.projectRequest.startDate),
      endDate: this.formatDate(this.projectRequest.endDate)
    };
    this.showConfirmModal = true;
  }

  confirmAssignment(): void {
    this.showConfirmModal = false;
    
    this.pmService.assignProject(this.projectRequest, this.loginPmId).subscribe({
      next: (response: ProjectResponseDTO) => {
        console.log('Project assigned successfully:', response);
        this.assignedProject = response;
        this.closeModal();
        this.successMessage = `Project "${response.name}" assigned successfully to ${response.teamLeadName}!`;
        this.showSuccessModal = true;
      },
      error: (error) => {
        console.error('Error assigning project:', error);
        
        if (error.error && error.error.message) {
          this.showError(error.error.message);
        } else if (error.status === 400) {
          this.showError('Invalid project data. Please check all fields and try again.');
        } else if (error.status === 404) {
          this.showError('Team Lead not found. Please select a valid Team Lead.');
        } else {
          this.showError('Failed to assign project. Please try again later.');
        }
      }
    });
  }

  cancelConfirmation(): void {
    this.showConfirmModal = false;
    this.confirmationData = null;
  }

  closeSuccessModal(): void {
    this.showSuccessModal = false;
    this.successMessage = '';
  }

  closeErrorModal(): void {
    this.showErrorModal = false;
    this.errorMessage = '';
  }

  showError(message: string): void {
    this.errorMessage = message;
    this.showErrorModal = true;
  }

  validateForm(): boolean {
    if (!this.projectRequest.name || this.projectRequest.name.trim() === '') {
      this.showError('Please enter a Project Name');
      return false;
    }

    if (this.projectRequest.budgetBandMin > this.projectRequest.budgetBandMax) {
      this.showError('Budget Min cannot be greater than Budget Max');
      return false;
    }

    if (this.projectRequest.budgetBandMin < 20 || this.projectRequest.budgetBandMin > 100) {
      this.showError('Budget Min should be between 20 and 100 Lakhs');
      return false;
    }

    if (this.projectRequest.budgetBandMax < 20 || this.projectRequest.budgetBandMax > 100) {
      this.showError('Budget Max should be between 20 and 100 Lakhs');
      return false;
    }
    
    const today = new Date();
    today.setHours(0, 0, 0, 0);
    const startDate = new Date(this.projectRequest.startDate);
    
    if (startDate < today) {
      this.showError('Start Date should be today or a future date');
      return false;
    }

    const endDate = new Date(this.projectRequest.endDate);
    if (endDate <= startDate) {
      this.showError('End Date must be after Start Date');
      return false;
    }

    if (!this.projectRequest.teamLeadId || this.projectRequest.teamLeadId === '') {
      this.showError('Please select a Team Lead');
      return false;
    }
    
    return true;
  }

  getStatusBadgeClass(status: string): string {
    switch (status) {
      case 'ACTIVE':
        return 'badge-ready';
      case 'PENDING':
        return 'badge-progress';
      case 'CLOSED':
        return 'badge-closed';
      default:
        return 'badge-progress';
    }
  }

  formatStatus(status: string): string {
    return status.replace(/_/g, ' ');
  }

  formatDate(dateString: string): string {
    if (!dateString) return '';
    const date = new Date(dateString);
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}-${month}-${day}`;
  }
}



CSS
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.2s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Modal Container */
.modal-container {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  max-width: 500px;
  width: 90%;
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Modal Header */
.modal-header {
  padding: 20px 24px;
  border-bottom: 1px solid #e5e7eb;
}

.modal-header h2 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
}

/* Success Header */
.success-header {
  background-color: #ecfdf5;
  border-bottom: 1px solid #a7f3d0;
  text-align: center;
}

.success-header h2 {
  color: #065f46;
}

.success-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  background-color: #10b981;
  color: white;
  border-radius: 50%;
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 12px;
}

/* Error Header */
.error-header {
  background-color: #fef2f2;
  border-bottom: 1px solid #fecaca;
  text-align: center;
}

.error-header h2 {
  color: #991b1b;
}

.error-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  background-color: #ef4444;
  color: white;
  border-radius: 50%;
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 12px;
}

/* Modal Body */
.modal-body {
  padding: 24px;
}

.modal-body p {
  margin: 0 0 16px 0;
  color: #4b5563;
  line-height: 1.5;
}

.success-modal .modal-body p,
.error-modal .modal-body p {
  text-align: center;
  font-size: 1.1rem;
}

/* Confirmation Details */
.confirmation-details {
  background-color: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 16px;
  margin-top: 16px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #e5e7eb;
}

.detail-row:last-child {
  border-bottom: none;
}

.detail-label {
  font-weight: 600;
  color: #6b7280;
}

.detail-value {
  color: #1f2937;
  font-weight: 500;
}

/* Modal Footer */
.modal-footer {
  padding: 16px 24px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.success-modal .modal-footer,
.error-modal .modal-footer {
  justify-content: center;
}

/* Buttons */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-primary {
  background-color: #2563eb;
  color: white;
}

.btn-primary:hover {
  background-color: #1d4ed8;
}

.btn-secondary {
  background-color: #f3f4f6;
  color: #374151;
}

.btn-secondary:hover {
  background-color: #e5e7eb;
}




Bench EMployee HTNml
<!-- Add these modals before the closing </body> tag in bench-employees.html -->

<!-- Confirmation Modal -->
<div *ngIf="showConfirmModal" class="modal-overlay">
  <div class="modal-container confirmation-modal">
    <div class="modal-header">
      <h2>Confirm Project Assignment</h2>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to assign this project to the following employee?</p>
      <div class="confirmation-details">
        <div class="detail-row">
          <span class="detail-label">Employee ID:</span>
          <span class="detail-value">{{ selectedEmployeeId }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Employee Name:</span>
          <span class="detail-value">{{ selectedEmployeeName }}</span>
        </div>
        <div class="detail-row" *ngIf="projectId">
          <span class="detail-label">Project ID:</span>
          <span class="detail-value">{{ projectId }}</span>
        </div>
        <div class="detail-row" *ngIf="jobRequestId">
          <span class="detail-label">Job Request ID:</span>
          <span class="detail-value">{{ jobRequestId }}</span>
        </div>
      </div>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="cancelConfirmation()">
        <i class="fas fa-times"></i> Cancel
      </button>
      <button type="button" class="btn btn-primary" (click)="confirmAssignment()">
        <i class="fas fa-check"></i> Confirm Assignment
      </button>
    </div>
  </div>
</div>

<!-- Success Modal -->
<div *ngIf="showSuccessModal" class="modal-overlay">
  <div class="modal-container success-modal">
    <div class="modal-header success-header">
      <div class="success-icon">
        <i class="fas fa-check-circle"></i>
      </div>
      <h2>Success!</h2>
    </div>
    <div class="modal-body">
      <p>{{ successMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeSuccessModal()">
        <i class="fas fa-thumbs-up"></i> OK
      </button>
    </div>
  </div>
</div>

<!-- Error Modal -->
<div *ngIf="showErrorModal" class="modal-overlay">
  <div class="modal-container error-modal">
    <div class="modal-header error-header">
      <div class="error-icon">
        <i class="fas fa-exclamation-circle"></i>
      </div>
      <h2>Error</h2>
    </div>
    <div class="modal-body">
      <p>{{ errorMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeErrorModal()">
        <i class="fas fa-times"></i> Close
      </button>
    </div>
  </div>
</div>





Bench Employee.ts
import { Component, OnInit } from '@angular/core';
import { BenchEmployeeDTO } from '../../dto/BenchEmployeeDTO';
import { PmService } from '../../service/pm-service';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-bench-employees',
  imports: [FormsModule, CommonModule],
  templateUrl: './bench-employees.html',
  styleUrl: './bench-employees.css'
})
export class BenchEmployees implements OnInit {

  benchEmployeeDto: BenchEmployeeDTO[] = [];
  filteredEmployees: BenchEmployeeDTO[] = [];

  skillsInput: string = '';
  minExperienceInput: number | null = null;
  roleFilterInput: string = '';

  jobRequestId: number = 0;
  projectId: number = 0;

  // Modal states
  showSuccessModal: boolean = false;
  showErrorModal: boolean = false;
  showConfirmModal: boolean = false;
  
  successMessage: string = '';
  errorMessage: string = '';
  
  // For confirmation modal
  selectedEmployeeId: string = '';
  selectedEmployeeName: string = '';

  constructor(private pmService: PmService, private route: ActivatedRoute) { }

  ngOnInit(): void {
    this.getAllEmployeesOnBench();
    this.route.queryParams.subscribe(params => {
      this.jobRequestId = params['jobRequestId'];
      this.projectId = params['projectId'];
    });
  }

  getAllEmployeesOnBench() {
    this.pmService.getAllEmployeesOnBench().subscribe({
      next: (data) => {
        console.log(data);
        this.benchEmployeeDto = data;
        this.filteredEmployees = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
        this.showError('Failed to load bench employees. Please try again.');
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  applyFilters() {
    const skillsLower = this.skillsInput.toLowerCase().trim();
    const skillFilters = skillsLower ? skillsLower.split(',').map(s => s.trim()) : [];

    this.filteredEmployees = this.benchEmployeeDto.filter(emp => {
      const empSkills = emp.skills.toLowerCase().split(',').map(s => s.trim());
      const empRole = emp.profileRole.toLowerCase().trim();

      // Check skills filter (all skillFilters must be included in empSkills)
      const skillsMatch = skillFilters.length === 0 ||
        skillFilters.every(skill => empSkills.includes(skill));

      // Check experience filter
      const expMatch = this.minExperienceInput === null ||
        emp.experience >= this.minExperienceInput;

      // Check role filter
      const roleMatch = !this.roleFilterInput ||
        empRole === this.roleFilterInput.toLowerCase().trim();

      return skillsMatch && expMatch && roleMatch;
    });
  }

  assignProjectToBenchEmployee(employeeId: string) {
    // Find employee details for confirmation
    const employee = this.benchEmployeeDto.find(emp => emp.employeeId === employeeId);
    if (employee) {
      this.selectedEmployeeId = employeeId;
      this.selectedEmployeeName = employee.fullName;
      this.showConfirmModal = true;
    }
  }

  confirmAssignment() {
    this.showConfirmModal = false;
    
    console.log('Assigning project to employee:', this.selectedEmployeeId);
    console.log('Job Request ID:', this.jobRequestId);
    console.log('ProjectId:', this.projectId);

    this.pmService.assignProjectToBenchEmployee(this.jobRequestId, this.projectId, this.selectedEmployeeId).subscribe(
      (response: string) => {
        console.log(response);
        this.showSuccess(response);
      },
      (error) => {
        console.error('Error assigning project:', error);
        const errorMsg = error.error?.message || 'Failed to assign project. Please try again.';
        this.showError(errorMsg);
      }
    );
  }

  cancelConfirmation() {
    this.showConfirmModal = false;
    this.selectedEmployeeId = '';
    this.selectedEmployeeName = '';
  }

  showSuccess(message: string) {
    this.successMessage = message;
    this.showSuccessModal = true;
  }

  showError(message: string) {
    this.errorMessage = message;
    this.showErrorModal = true;
  }

  closeSuccessModal() {
    this.showSuccessModal = false;
    this.successMessage = '';
    // Optionally refresh the employee list
    this.getAllEmployeesOnBench();
  }

  closeErrorModal() {
    this.showErrorModal = false;
    this.errorMessage = '';
  }

  getInitials(fullName: string): string {
    return fullName
      .split(' ')
      .map(name => name.charAt(0))
      .join('')
      .toUpperCase();
  }

  trackByEmployeeId(index: number, employee: BenchEmployeeDTO): string {
    return employee.employeeId;
  }
}



Bench EMPloyee.css 
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  animation: fadeIn 0.2s ease-in;
  backdrop-filter: blur(2px);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Modal Container */
.modal-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
  max-width: 520px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    transform: translateY(30px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Modal Header */
.modal-header {
  padding: 24px 28px;
  border-bottom: 1px solid #e5e7eb;
}

.modal-header h2 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
}

/* Success Header */
.success-header {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  border-bottom: none;
  text-align: center;
  padding: 32px 28px;
}

.success-header h2 {
  color: white;
  margin-top: 12px;
}

.success-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 64px;
  height: 64px;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  border-radius: 50%;
  font-size: 36px;
  margin-bottom: 8px;
  animation: scaleIn 0.4s ease-out;
}

@keyframes scaleIn {
  from {
    transform: scale(0);
  }
  to {
    transform: scale(1);
  }
}

/* Error Header */
.error-header {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  border-bottom: none;
  text-align: center;
  padding: 32px 28px;
}

.error-header h2 {
  color: white;
  margin-top: 12px;
}

.error-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 64px;
  height: 64px;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  border-radius: 50%;
  font-size: 36px;
  margin-bottom: 8px;
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}

/* Modal Body */
.modal-body {
  padding: 28px;
}

.modal-body p {
  margin: 0 0 20px 0;
  color: #4b5563;
  line-height: 1.6;
  font-size: 1rem;
}

.success-modal .modal-body p,
.error-modal .modal-body p {
  text-align: center;
  font-size: 1.1rem;
  color: #1f2937;
  font-weight: 500;
}

/* Confirmation Details */
.confirmation-details {
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  margin-top: 20px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #e5e7eb;
}

.detail-row:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

.detail-row:first-child {
  padding-top: 0;
}

.detail-label {
  font-weight: 600;
  color: #6b7280;
  font-size: 0.95rem;
}

.detail-value {
  color: #1f2937;
  font-weight: 600;
  font-size: 1rem;
}

/* Modal Footer */
.modal-footer {
  padding: 20px 28px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  background-color: #f9fafb;
  border-radius: 0 0 12px 12px;
}

.success-modal .modal-footer,
.error-modal .modal-footer {
  justify-content: center;
  background-color: white;
}

/* Buttons */
.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.btn i {
  font-size: 14px;
}

.btn-primary {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  box-shadow: 0 2px 8px rgba(37, 99, 235, 0.3);
}

.btn-primary:hover {
  background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.4);
}

.btn-primary:active {
  transform: translateY(0);
}

.btn-secondary {
  background-color: white;
  color: #374151;
  border: 2px solid #d1d5db;
}

.btn-secondary:hover {
  background-color: #f9fafb;
  border-color: #9ca3af;
}

.btn-secondary:active {
  background-color: #f3f4f6;
}

/* Success/Error specific button styles */
.success-modal .btn-primary {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);
}

.success-modal .btn-primary:hover {
  background: linear-gradient(135deg, #059669 0%, #047857 100%);
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
}

.error-modal .btn-primary {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
}

.error-modal .btn-primary:hover {
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
}

/* Responsive Design */
@media (max-width: 600px) {
  .modal-container {
    width: 95%;
    margin: 20px;
  }

  .modal-header,
  .modal-body,
  .modal-footer {
    padding: 20px;
  }

  .detail-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .btn {
    width: 100%;
    justify-content: center;
  }

  .modal-footer {
    flex-direction: column-reverse;
  }
}


JObRequest OF HTML
<!-- Add these modals before the closing </body> tag in pm-job-requests.html -->
<!-- Confirmation Modal -->
<div *ngIf="showConfirmModal" class="modal-overlay-custom">
  <div class="modal-container-custom confirmation-modal-custom">
    <div class="modal-header-custom">
      <div class="modal-icon-custom" [ngClass]="confirmAction === 'decline' ? 'warning-icon' : 'info-icon'">
        <i [class]="confirmAction === 'decline' ? 'fas fa-exclamation-triangle' : 'fas fa-question-circle'"></i>
      </div>
      <h2>{{ getConfirmationTitle() }}</h2>
    </div>
    <div class="modal-body-custom">
      <p>{{ getConfirmationMessage() }}</p>
      
      <!-- Show details for forward action -->
      <div *ngIf="confirmAction === 'forward'" class="confirmation-details-custom">
        <div class="detail-row-custom">
          <span class="detail-label-custom">HR ID:</span>
          <span class="detail-value-custom">{{ hrForwardDTO.hrId }}</span>
        </div>
        <div class="detail-row-custom">
          <span class="detail-label-custom">Priority:</span>
          <span class="detail-value-custom" [ngClass]="getPriorityClass(hrForwardDTO.priority)">
            {{ hrForwardDTO.priority }}
          </span>
        </div>
        <div class="detail-row-custom">
          <span class="detail-label-custom">CTC Range:</span>
          <span class="detail-value-custom">₹{{ hrForwardDTO.minCtc }}L - ₹{{ hrForwardDTO.maxCtc }}L</span>
        </div>
      </div>
    </div>
    <div class="modal-footer-custom">
      <button type="button" class="btn-custom btn-secondary-custom" (click)="cancelConfirmation()">
        <i class="fas fa-times"></i> Cancel
      </button>
      <button 
        type="button" 
        class="btn-custom"
        [ngClass]="confirmAction === 'decline' ? 'btn-danger-custom' : 'btn-primary-custom'"
        (click)="confirmModalAction()"
      >
        <i [class]="confirmAction === 'decline' ? 'fas fa-ban' : 'fas fa-paper-plane'"></i>
        {{ confirmAction === 'decline' ? 'Decline' : 'Forward' }}
      </button>
    </div>
  </div>
</div>

<!-- Success Modal -->
<div *ngIf="showSuccessModal" class="modal-overlay-custom">
  <div class="modal-container-custom success-modal-custom">
    <div class="modal-header-custom success-header-custom">
      <div class="success-icon-custom">
        <i class="fas fa-check-circle"></i>
      </div>
      <h2>Success!</h2>
    </div>
    <div class="modal-body-custom">
      <p>{{ successMessage }}</p>
    </div>
    <div class="modal-footer-custom">
      <button type="button" class="btn-custom btn-primary-custom" (click)="closeSuccessModal()">
        <i class="fas fa-thumbs-up"></i> OK
      </button>
    </div>
  </div>
</div>

<!-- Error Modal -->
<div *ngIf="showErrorModal" class="modal-overlay-custom">
  <div class="modal-container-custom error-modal-custom">
    <div class="modal-header-custom error-header-custom">
      <div class="error-icon-custom">
        <i class="fas fa-times-circle"></i>
      </div>
      <h2>Error</h2>
    </div>
    <div class="modal-body-custom">
      <p>{{ errorMessage }}</p>
    </div>
    <div class="modal-footer-custom">
      <button type="button" class="btn-custom btn-primary-custom" (click)="closeErrorModal()">
        <i class="fas fa-redo"></i> Close
      </button>
    </div>
  </div>
</div>


JOb Request OF PM  TS 
import { ChangeDetectorRef, Component, OnInit } from '@angular/core';
import { Router, RouterLink } from '@angular/router';
import { PmService } from '../../service/pm-service';
import { PmJobRequestResponseDTO } from '../../dto/PmJobRequestResponseDTO';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { HrForwardDTO } from '../../dto/HrForwardDTO';

@Component({
  selector: 'app-pm-job-requests',
  imports: [RouterLink, FormsModule, CommonModule],
  templateUrl: './pm-job-requests.html',
  styleUrl: './pm-job-requests.css'
})
export class PmJobRequests implements OnInit {

  countOfAllJobRequests: number = 0;
  countOfPendingJobRequests: number = 0;
  countOfApprovedJobRequests: number = 0;
  CountOfClosedJobRequests: number = 0;
  countOfDeclinedJobRequests: number = 0;

  pmJobResponse: PmJobRequestResponseDTO[] = []; 
  filteredJobRequests: PmJobRequestResponseDTO[] = []; 

  selectedFilter: string = 'all';
  hrForwardDTO: HrForwardDTO = new HrForwardDTO(); 
  showHrForm: boolean = false;
  selectedJobRequestId: number | null = null;

  hrIds: string[] = [];
  
  // Modal states
  showSuccessModal: boolean = false;
  showErrorModal: boolean = false;
  showConfirmModal: boolean = false;
  
  successMessage: string = '';
  errorMessage: string = '';
  
  // For decline confirmation
  confirmAction: 'decline' | 'forward' | null = null;
  pendingJobRequestId: number | null = null;

  loginPmId: string = "EMP1002";

  constructor(private pmService: PmService, private router: Router) { }

  ngOnInit(): void {
    this.getCountOfAllJObRequests(this.loginPmId);
    this.getCountOfPendingRequest(this.loginPmId);
    this.getCountOfClosedJobRequests(this.loginPmId);
    this.getCountOfApprovedJobRequests(this.loginPmId);
    this.getCountOfDeclinedJobRequests(this.loginPmId);
    this.getAllJobrequestsByPmId(this.loginPmId);
    this.applyFilter('all');
    this.fetchHrIds();
  }

  getCountOfAllJObRequests(loginPmId: string) {
    this.pmService.getCountOfAllJObRequests(loginPmId).subscribe({
      next: (data: number) => {
        console.log('Count OF Total JOb Requests From API', data);
        this.countOfAllJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getCountOfPendingRequest(loginPmId: string) {
    this.pmService.getCountOfPendingRequest(loginPmId).subscribe({
      next: (data: number) => {
        console.log('Count OF Pending Job Requests From API', data);
        this.countOfPendingJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getCountOfApprovedJobRequests(loginPmId: string) {
    this.pmService.getCountOfApprovedJobRequests(loginPmId).subscribe({
      next: (data: number) => {
        console.log('Count OF Forwarded To Hr From API', data);
        this.countOfApprovedJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getCountOfClosedJobRequests(loginPmId: string) {
    this.pmService.getCountOfClosedJobRequests(loginPmId).subscribe({
      next: (data: number) => {
        console.log('Count OF Assigned to Bench From API', data);
        this.CountOfClosedJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getCountOfDeclinedJobRequests(loginPmId: string) {
    this.pmService.getCountOfDeclinedJobRequests(loginPmId).subscribe({
      next: (data: number) => {
        console.log('Count OF Declined From API', data);
        this.countOfDeclinedJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getAllJobrequestsByPmId(loginPmId: string) {
    this.pmService.getAllJobRequests(loginPmId).subscribe({
      next: (data) => {
        console.log('All JOb Request From API ', data);
        this.pmJobResponse = data;
        this.filteredJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getStatusClass(status: string): string {
    switch (status) {
      case 'SUBMITTED':
        return 'status-submitted';
      case 'FORWARDED_TO_HR':
        return 'status-forwarded';
      case 'CLOSED':
        return 'status-closed';
      case 'DECLINED':
        return 'status-declined';
      default:
        return '';
    }
  }

  statusLabelMap: { [key: string]: string } = {
    'SUBMITTED': 'Pending',
    'FORWARDED_TO_HR': 'Forwarded To HR',
    'CLOSED': 'Assigned To Bench'
  };

  getStatusLabel(status: string): string {
    return this.statusLabelMap[status] || status;
  }

  getPriorityClass(priority: string): string {
    switch (priority?.toUpperCase()) {
      case 'LOW':
        return 'priority-low';
      case 'MEDIUM':
        return 'priority-medium';
      case 'HIGH':
        return 'priority-high';
      default:
        return '';
    }
  }

  applyFilter(filter: string) {
    this.selectedFilter = filter;

    switch (filter) {
      case 'assigned':
        this.filteredJobRequests = this.pmJobResponse.filter(job => job.status === 'CLOSED');
        break;
      case 'forwarded':
        this.filteredJobRequests = this.pmJobResponse.filter(job => job.status === 'FORWARDED_TO_HR');
        break;
      case 'pending':
        this.filteredJobRequests = this.pmJobResponse.filter(job => job.status === 'SUBMITTED');
        break;
      case 'declined':
        this.filteredJobRequests = this.pmJobResponse.filter(job => job.status === 'DECLINED');
        break;
      case 'all':
      default:
        this.filteredJobRequests = [...this.pmJobResponse];
    }
  }

  navigateToBench(jobRequestId: number, projectId: number) {
    this.router.navigate(['/pm-dashboard/bench-employees'], {
      queryParams: { jobRequestId, projectId }
    });
  }

  fetchHrIds(): void {
    this.pmService.findHrIDs().subscribe({
      next: (ids: string[]) => {
        this.hrIds = ids;
        console.log('Hr IDs fetched:', ids);
      },
      error: (error) => {
        console.error('Error fetching HR IDs:', error);
        this.showError('Failed to fetch HR IDs. Please try again.');
      }
    });
  }

  openForwardToHRForm(jobRequestId: number): void {
    this.selectedJobRequestId = jobRequestId;
    this.showHrForm = true;

    const selectedJob = this.pmJobResponse.find(job => job.jobRequestId === jobRequestId);
    if (selectedJob) {
      this.hrForwardDTO = {
        hrId: '',
        priority: selectedJob.priority,
        minCtc: selectedJob.minCtc,
        maxCtc: selectedJob.maxCtc,
      };
    }
  }

  closeForwardToHRForm(): void {
    this.showHrForm = false;
  }

  onSubmitForwardToHR(): void {
    if (!this.hrForwardDTO.hrId) {
      this.showError('Please select an HR ID.');
      return;
    }

    if (this.hrForwardDTO.minCtc >= this.hrForwardDTO.maxCtc) {
      this.showError('Min CTC must be less than Max CTC.');
      return;
    }

    const jobRequestId = this.selectedJobRequestId;
    if (jobRequestId) {
      this.confirmAction = 'forward';
      this.pendingJobRequestId = jobRequestId;
      this.showConfirmModal = true;
    }
  }

  forwardToHr(hrForwardDTO: HrForwardDTO, jobRequestId: number): void {
    this.pmService.ForwardTohr(hrForwardDTO, jobRequestId).subscribe({
      next: (data) => {
        console.log('Successfully forwarded to HR:', data);
        this.closeForwardToHRForm();
        this.showSuccess('Job request forwarded to HR successfully!');
        this.refreshData();
      },
      error: (error) => {
        console.error('Error forwarding to HR:', error);
        const errorMsg = error.error || 'Failed to forward job request to HR.';
        this.showError(errorMsg);
      }
    });
  }

  declineJobRequest(jobRequestId: number): void {
    this.confirmAction = 'decline';
    this.pendingJobRequestId = jobRequestId;
    this.showConfirmModal = true;
  }

  performDecline(jobRequestId: number): void {
    this.pmService.declineJoBRequest(jobRequestId).subscribe({
      next: (data: string) => {
        console.log('Job Request Declined Successfully with id:', jobRequestId);
        this.showSuccess(data);
        this.refreshData();
      },
      error: (error) => {
        console.error('Error declining the request:', error);
        const errorMsg = error.error || 'Failed to decline job request.';
        this.showError(errorMsg);
      }
    });
  }

  confirmModalAction(): void {
    this.showConfirmModal = false;

    if (this.confirmAction === 'decline' && this.pendingJobRequestId !== null) {
      this.performDecline(this.pendingJobRequestId);
    } else if (this.confirmAction === 'forward' && this.pendingJobRequestId !== null) {
      this.forwardToHr(this.hrForwardDTO, this.pendingJobRequestId);
    }

    this.confirmAction = null;
    this.pendingJobRequestId = null;
  }

  cancelConfirmation(): void {
    this.showConfirmModal = false;
    this.confirmAction = null;
    this.pendingJobRequestId = null;
  }

  showSuccess(message: string): void {
    this.successMessage = message;
    this.showSuccessModal = true;
  }

  showError(message: string): void {
    this.errorMessage = message;
    this.showErrorModal = true;
  }

  closeSuccessModal(): void {
    this.showSuccessModal = false;
    this.successMessage = '';
  }

  closeErrorModal(): void {
    this.showErrorModal = false;
    this.errorMessage = '';
  }

  refreshData(): void {
    this.getCountOfAllJObRequests(this.loginPmId);
    this.getCountOfPendingRequest(this.loginPmId);
    this.getCountOfClosedJobRequests(this.loginPmId);
    this.getCountOfApprovedJobRequests(this.loginPmId);
    this.getCountOfDeclinedJobRequests(this.loginPmId);
    this.getAllJobrequestsByPmId(this.loginPmId);
  }

  getConfirmationMessage(): string {
    if (this.confirmAction === 'decline') {
      return 'Are you sure you want to decline this job request? This action cannot be undone.';
    } else if (this.confirmAction === 'forward') {
      const job = this.pmJobResponse.find(j => j.jobRequestId === this.pendingJobRequestId);
      return `Are you sure you want to forward job request "${job?.title}" to HR ${this.hrForwardDTO.hrId}?`;
    }
    return '';
  }

  getConfirmationTitle(): string {
    if (this.confirmAction === 'decline') {
      return 'Decline Job Request';
    } else if (this.confirmAction === 'forward') {
      return 'Forward to HR';
    }
    return 'Confirm Action';
  }
}





Jobrequests of pm CSS 

/* Custom Modal Styles to avoid conflicts */
.modal-overlay-custom {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  animation: fadeInCustom 0.2s ease-in;
  backdrop-filter: blur(3px);
}

@keyframes fadeInCustom {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-container-custom {
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.25);
  max-width: 520px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUpCustom 0.3s ease-out;
}

@keyframes slideUpCustom {
  from {
    transform: translateY(30px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Modal Header */
.modal-header-custom {
  padding: 28px;
  border-bottom: 1px solid #e5e7eb;
  text-align: center;
}

.modal-header-custom h2 {
  margin: 12px 0 0 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
}

/* Modal Icons */
.modal-icon-custom {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 64px;
  height: 64px;
  border-radius: 50%;
  font-size: 32px;
  margin-bottom: 8px;
}

.warning-icon {
  background-color: #fef3c7;
  color: #f59e0b;
  animation: pulseCustom 2s infinite;
}

.info-icon {
  background-color: #dbeafe;
  color: #3b82f6;
}

@keyframes pulseCustom {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

/* Success Header */
.success-header-custom {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  border-bottom: none;
  padding: 32px 28px;
}

.success-header-custom h2 {
  color: white;
}

.success-icon-custom {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 72px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 40px;
  margin-bottom: 12px;
  animation: successBounce 0.6s ease-out;
}

@keyframes successBounce {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

/* Error Header */
.error-header-custom {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  border-bottom: none;
  padding: 32px 28px;
}

.error-header-custom h2 {
  color: white;
}

.error-icon-custom {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 72px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 40px;
  margin-bottom: 12px;
  animation: errorShake 0.5s ease-in-out;
}

@keyframes errorShake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-8px); }
  75% { transform: translateX(8px); }
}

/* Modal Body */
.modal-body-custom {
  padding: 28px;
}

.modal-body-custom p {
  margin: 0 0 20px 0;
  color: #4b5563;
  line-height: 1.6;
  font-size: 1.05rem;
  text-align: center;
}

.success-modal-custom .modal-body-custom p,
.error-modal-custom .modal-body-custom p {
  font-size: 1.1rem;
  color: #1f2937;
  font-weight: 500;
}

/* Confirmation Details */
.confirmation-details-custom {
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  margin-top: 20px;
  text-align: left;
}

.detail-row-custom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #e5e7eb;
}

.detail-row-custom:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

.detail-row-custom:first-child {
  padding-top: 0;
}

.detail-label-custom {
  font-weight: 600;
  color: #6b7280;
  font-size: 0.95rem;
}

.detail-value-custom {
  color: #1f2937;
  font-weight: 600;
  font-size: 1rem;
}

/* Modal Footer */
.modal-footer-custom {
  padding: 20px 28px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: center;
  gap: 12px;
  background-color: #f9fafb;
  border-radius: 0 0 12px 12px;
}

/* Buttons */
.btn-custom {
  padding: 12px 28px;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  min-width: 120px;
  justify-content: center;
}

.btn-custom i {
  font-size: 14px;
}

.btn-primary-custom {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  box-shadow: 0 2px 8px rgba(37, 99, 235, 0.3);
}

.btn-primary-custom:hover {
  background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.4);
}

.btn-secondary-custom {
  background-color: white;
  color: #374151;
  border: 2px solid #d1d5db;
}

.btn-secondary-custom:hover {
  background-color: #f9fafb;
  border-color: #9ca3af;
}

.btn-danger-custom {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: white;
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
}

.btn-danger-custom:hover {
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
}

.btn-custom:active {
  transform: translateY(0);
}

/* Priority badges in confirmation */
.confirmation-details-custom .priority-low {
  color: #10b981;
  font-weight: 700;
}

.confirmation-details-custom .priority-medium {
  color: #f59e0b;
  font-weight: 700;
}

.confirmation-details-custom .priority-high {
  color: #ef4444;
  font-weight: 700;
}

/* Responsive Design */
@media (max-width: 600px) {
  .modal-container-custom {
    width: 95%;
    margin: 20px;
  }

  .modal-header-custom,
  .modal-body-custom,
  .modal-footer-custom {
    padding: 20px;
  }

  .detail-row-custom {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .btn-custom {
    width: 100%;
    min-width: unset;
  }

  .modal-footer-custom {
    flex-direction: column-reverse;
  }

  .modal-icon-custom,
  .success-icon-custom,
  .error-icon-custom {
    width: 56px;
    height: 56px;
    font-size: 28px;
  }
}





hr job request TS 
import { Component, OnInit } from '@angular/core';
import { HrService } from '../../service/hr-service';
import { PmJobRequestResponseDTO } from '../../dto/PmJobRequestResponseDTO';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-hr-job-requests',
  imports: [FormsModule, CommonModule],
  templateUrl: './hr-job-requests.html',
  styleUrl: './hr-job-requests.css'
})
export class HrJobRequests implements OnInit {

  countOfjobRequests: number = 0;
  countOfPostedJobs: number = 0;
  countOfPendingJobRequests: number = 0;
  selectedFilter: string = 'all';

  JobResponse: PmJobRequestResponseDTO[] = [];
  filteredJobRequests: PmJobRequestResponseDTO[] = [];

  loginHrId: string = 'EMP1003';
  
  // Modal states
  showSuccessModal: boolean = false;
  showErrorModal: boolean = false;
  showConfirmModal: boolean = false;
  
  successMessage: string = '';
  errorMessage: string = '';
  
  // For post job confirmation
  selectedJobRequestId: number | null = null;
  selectedJobTitle: string = '';
  
  constructor(private hrService: HrService) {}

  ngOnInit(): void {
    console.log('ngOnInit called with loginHrId:', this.loginHrId);
    console.log('Initial filteredJobRequests:', this.filteredJobRequests);
    
    this.getCountOfJobRequests(this.loginHrId);
    this.getCountOfPostedJobs(this.loginHrId);
    this.getCountOfPendingJobRequests(this.loginHrId);
    this.getAllJobrequestsByHrId(this.loginHrId);
  }

  getCountOfJobRequests(loginHrId: string): void {
    this.hrService.getCountOfJobRequests(loginHrId).subscribe({
      next: (data: number) => {
        console.log('Count OF Job Requests From API', data);
        this.countOfjobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      }
    });
  }
  
  getCountOfPostedJobs(loginHrId: string): void {
    this.hrService.getCountOfPostedJobs(loginHrId).subscribe({
      next: (data: number) => {
        console.log('Count OF Posted Jobs From API', data);
        this.countOfPostedJobs = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      }
    });
  }

  getCountOfPendingJobRequests(loginHrId: string): void {
    this.hrService.getCountOfPendingJobRequests(loginHrId).subscribe({
      next: (data: number) => {
        console.log('Count OF Pending Job Requests From API', data);
        this.countOfPendingJobRequests = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
      }
    });
  }

  getAllJobrequestsByHrId(loginHrId: string): void {
    this.hrService.getAllJobRequests(loginHrId).subscribe({
      next: (data: PmJobRequestResponseDTO[]) => {
        console.log('All Job Requests From API', data);
        this.JobResponse = Array.isArray(data) ? data : [];
        this.filteredJobRequests = Array.isArray(data) ? [...data] : [];
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
        this.JobResponse = [];
        this.filteredJobRequests = [];
        this.showError('Failed to load job requests. Please try again.');
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  getStatusClass(status: string): string {
    switch (status) {
      case 'POSTED':
        return 'status-posted';
      case 'FORWARDED_TO_HR':
        return 'status-forwarded';
      default:
        return '';
    }
  }

  statusLabelMap: { [key: string]: string } = {
    'POSTED': 'Posted',
    'FORWARDED_TO_HR': 'Pending',
  };

  getStatusLabel(status: string): string {
    return this.statusLabelMap[status] || status;
  }

  getPriorityClass(priority: string): string {
    switch (priority?.toUpperCase()) {
      case 'LOW':
        return 'priority-low';
      case 'MEDIUM':
        return 'priority-medium';
      case 'HIGH':
        return 'priority-high';
      default:
        return '';
    }
  }

  applyFilter(filter: string): void {
    this.selectedFilter = filter;

    switch (filter) {
      case 'pending':
        this.filteredJobRequests = this.JobResponse.filter(job => job.status === 'FORWARDED_TO_HR');
        break;
      case 'posted':
        this.filteredJobRequests = this.JobResponse.filter(job => job.status === 'POSTED');
        break;
      case 'all':
      default:
        this.filteredJobRequests = [...this.JobResponse];
    }
  }

  postJob(jobRequestId: number): void {
    // Find job details for confirmation
    const job = this.JobResponse.find(j => j.jobRequestId === jobRequestId);
    if (job) {
      this.selectedJobRequestId = jobRequestId;
      this.selectedJobTitle = job.title;
      this.showConfirmModal = true;
    }
  }

  confirmPostJob(): void {
    this.showConfirmModal = false;
    
    if (this.selectedJobRequestId === null) return;

    this.hrService.postJob(this.selectedJobRequestId).subscribe({
      next: (data) => {
        console.log(data);
        this.showSuccess(data || 'Job posted successfully!');
        this.refreshData();
      },
      error: (err) => {
        console.error("Exception occurred while posting job", err);
        const errorMsg = err.error?.message || err.error || 'Failed to post job. Please try again.';
        this.showError(errorMsg);
      }
    });

    this.selectedJobRequestId = null;
    this.selectedJobTitle = '';
  }

  cancelConfirmation(): void {
    this.showConfirmModal = false;
    this.selectedJobRequestId = null;
    this.selectedJobTitle = '';
  }

  showSuccess(message: string): void {
    this.successMessage = message;
    this.showSuccessModal = true;
  }

  showError(message: string): void {
    this.errorMessage = message;
    this.showErrorModal = true;
  }

  closeSuccessModal(): void {
    this.showSuccessModal = false;
    this.successMessage = '';
  }

  closeErrorModal(): void {
    this.showErrorModal = false;
    this.errorMessage = '';
  }

  refreshData(): void {
    this.getCountOfJobRequests(this.loginHrId);
    this.getCountOfPostedJobs(this.loginHrId);
    this.getCountOfPendingJobRequests(this.loginHrId);
    this.getAllJobrequestsByHrId(this.loginHrId);
  }
}




hr job REquest HTML 
<!-- Add these modals before the closing </body> tag in hr-job-requests.html -->

<!-- Confirmation Modal -->
<div *ngIf="showConfirmModal" class="modal-overlay">
  <div class="modal-container confirmation-modal">
    <div class="modal-header">
      <div class="confirm-icon">
        <i class="fas fa-bullhorn"></i>
      </div>
      <h2>Confirm Job Posting</h2>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to post this job?</p>
      <div class="confirmation-details">
        <div class="detail-row">
          <span class="detail-label">Job Request ID:</span>
          <span class="detail-value">{{ selectedJobRequestId }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Job Title:</span>
          <span class="detail-value">{{ selectedJobTitle }}</span>
        </div>
      </div>
      <p class="confirmation-note">
        <i class="fas fa-info-circle"></i>
        This will make the job visible to external candidates.
      </p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="cancelConfirmation()">
        <i class="fas fa-times"></i> Cancel
      </button>
      <button type="button" class="btn btn-primary" (click)="confirmPostJob()">
        <i class="fas fa-paper-plane"></i> Post Job
      </button>
    </div>
  </div>
</div>

<!-- Success Modal -->
<div *ngIf="showSuccessModal" class="modal-overlay">
  <div class="modal-container success-modal">
    <div class="modal-header success-header">
      <div class="success-icon">
        <i class="fas fa-check-circle"></i>
      </div>
      <h2>Success!</h2>
    </div>
    <div class="modal-body">
      <p>{{ successMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeSuccessModal()">
        <i class="fas fa-thumbs-up"></i> OK
      </button>
    </div>
  </div>
</div>

<!-- Error Modal -->
<div *ngIf="showErrorModal" class="modal-overlay">
  <div class="modal-container error-modal">
    <div class="modal-header error-header">
      <div class="error-icon">
        <i class="fas fa-exclamation-circle"></i>
      </div>
      <h2>Error</h2>
    </div>
    <div class="modal-body">
      <p>{{ errorMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-primary" (click)="closeErrorModal()">
        <i class="fas fa-redo"></i> Try Again
      </button>
    </div>
  </div>
</div>



Hr job Request css 
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  animation: fadeIn 0.2s ease-in;
  backdrop-filter: blur(3px);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Modal Container */
.modal-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.25);
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    transform: translateY(30px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Modal Header */
.modal-header {
  padding: 28px;
  border-bottom: 1px solid #e5e7eb;
  text-align: center;
}

.modal-header h2 {
  margin: 12px 0 0 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
}

/* Confirm Icon */
.confirm-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 64px;
  height: 64px;
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  border-radius: 50%;
  font-size: 32px;
  margin-bottom: 8px;
  animation: bounceIn 0.5s ease-out;
}

@keyframes bounceIn {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

/* Success Header */
.success-header {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  border-bottom: none;
  padding: 32px 28px;
}

.success-header h2 {
  color: white;
}

.success-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 72px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 40px;
  margin-bottom: 12px;
  animation: successPop 0.6s ease-out;
}

@keyframes successPop {
  0% {
    transform: scale(0) rotate(0deg);
  }
  50% {
    transform: scale(1.2) rotate(180deg);
  }
  100% {
    transform: scale(1) rotate(360deg);
  }
}

/* Error Header */
.error-header {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  border-bottom: none;
  padding: 32px 28px;
}

.error-header h2 {
  color: white;
}

.error-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 72px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 40px;
  margin-bottom: 12px;
  animation: errorShake 0.5s ease-in-out;
}

@keyframes errorShake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-10px); }
  40% { transform: translateX(10px); }
  60% { transform: translateX(-8px); }
  80% { transform: translateX(8px); }
}

/* Modal Body */
.modal-body {
  padding: 28px;
}

.modal-body p {
  margin: 0 0 20px 0;
  color: #4b5563;
  line-height: 1.6;
  font-size: 1.05rem;
  text-align: center;
}

.success-modal .modal-body p,
.error-modal .modal-body p {
  font-size: 1.1rem;
  color: #1f2937;
  font-weight: 500;
}

/* Confirmation Details */
.confirmation-details {
  background: linear-gradient(135deg, #eff6ff 0%, #dbeafe 100%);
  border: 1px solid #bfdbfe;
  border-radius: 8px;
  padding: 20px;
  margin-top: 20px;
  text-align: left;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #bfdbfe;
}

.detail-row:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

.detail-row:first-child {
  padding-top: 0;
}

.detail-label {
  font-weight: 600;
  color: #1e40af;
  font-size: 0.95rem;
}

.detail-value {
  color: #1f2937;
  font-weight: 600;
  font-size: 1rem;
}

/* Confirmation Note */
.confirmation-note {
  margin-top: 20px;
  padding: 12px 16px;
  background-color: #fef3c7;
  border-left: 4px solid #f59e0b;
  border-radius: 4px;
  color: #92400e;
  font-size: 0.9rem;
  text-align: left;
}

.confirmation-note i {
  margin-right: 8px;
  color: #f59e0b;
}

/* Modal Footer */
.modal-footer {
  padding: 20px 28px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: center;
  gap: 12px;
  background-color: #f9fafb;
  border-radius: 0 0 12px 12px;
}

/* Buttons */
.btn {
  padding: 12px 28px;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  min-width: 120px;
  justify-content: center;
}

.btn i {
  font-size: 14px;
}

.btn-primary {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  box-shadow: 0 2px 8px rgba(37, 99, 235, 0.3);
}

.btn-primary:hover {
  background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.4);
}

.btn-secondary {
  background-color: white;
  color: #374151;
  border: 2px solid #d1d5db;
}

.btn-secondary:hover {
  background-color: #f9fafb;
  border-color: #9ca3af;
  transform: translateY(-1px);
}

.btn:active {
  transform: translateY(0);
}

/* Success Modal Button */
.success-modal .btn-primary {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);
}

.success-modal .btn-primary:hover {
  background: linear-gradient(135deg, #059669 0%, #047857 100%);
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
}

/* Error Modal Button */
.error-modal .btn-primary {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
}

.error-modal .btn-primary:hover {
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
}

/* Responsive Design */
@media (max-width: 600px) {
  .modal-container {
    width: 95%;
    margin: 20px;
  }

  .modal-header,
  .modal-body,
  .modal-footer {
    padding: 20px;
  }

  .detail-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .btn {
    width: 100%;
    min-width: unset;
  }

  .modal-footer {
    flex-direction: column-reverse;
  }

  .confirm-icon,
  .success-icon,
  .error-icon {
    width: 56px;
    height: 56px;
    font-size: 28px;
  }

  .modal-header h2 {
    font-size: 1.3rem;
  }

  .modal-body p {
    font-size: 1rem;
  }
}


Applied canddidates .ts 
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { AppliedCandidateDTO } from '../../dto/AppliedCandidateDTO';
import { HrService } from '../../service/hr-service';
import { CandidateFilterRequestDTO } from '../../dto/CandidateFilterRequestDTO';

@Component({
  selector: 'app-applied-candidates',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './applied-candidates.html',
  styleUrl: './applied-candidates.css'
})
export class AppliedCandidates implements OnInit {
  
  allCandidates: AppliedCandidateDTO[] = [];
  displayedCandidates: AppliedCandidateDTO[] = [];
  loginHrId: string = 'EMP1003';
  
  // Filter properties
  selectedExperience: string = '';
  minSalary: number | null = null;
  maxSalary: number | null = null;
  selectedNoticePeriod: number | null = null;
  selectedSkills: string = '';

  shortlistedStatus: string = "SHORTLISTED";
  notShortlistedStatus: string = "NOT_SHORTLISTED";

  // Modal states
  showSuccessModal: boolean = false;
  showErrorModal: boolean = false;
  showConfirmModal: boolean = false;
  
  successMessage: string = '';
  errorMessage: string = '';
  
  // For confirmation
  confirmAction: 'shortlist' | 'reject' | null = null;
  selectedApplicationId: number | null = null;
  selectedCandidateName: string = '';

  ngOnInit(): void {
    this.getAllCandidates(this.loginHrId);
  }

  constructor(private hrService: HrService) {}

  getAllCandidates(loginHrId: string) {
    this.hrService.getAllCandidates(loginHrId).subscribe({
      next: (data) => {
        console.log('All Candidates From API ', data);
        this.allCandidates = data;
        this.displayedCandidates = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling API", err);
        this.showError('Failed to load candidates. Please try again.');
      },
      complete: () => {
        console.log("Data retrieval completed successfully");
      }
    });
  }

  applyFilters() {
    const filterRequest = new CandidateFilterRequestDTO();
    
    // Set experience range based on selection
    if (this.selectedExperience === 'entry') {
      filterRequest.minExperience = 0;
      filterRequest.maxExperience = 2;
    } else if (this.selectedExperience === 'mid') {
      filterRequest.minExperience = 2;
      filterRequest.maxExperience = 5;
    } else if (this.selectedExperience === 'senior') {
      filterRequest.minExperience = 5;
      filterRequest.maxExperience = 8;
    } else if (this.selectedExperience === 'lead') {
      filterRequest.minExperience = 8;
      filterRequest.maxExperience = 100;
    } else {
      filterRequest.minExperience = null;
      filterRequest.maxExperience = null;
    }
    
    // Set salary range
    filterRequest.minExpectedCtc = this.minSalary;
    filterRequest.maxExpectedCtc = this.maxSalary;

    // Set notice period
    filterRequest.noticePeriod = this.selectedNoticePeriod;

    // Set skills
    if (this.selectedSkills && this.selectedSkills.trim().length > 0) {
      filterRequest.skills = this.selectedSkills.split(',').map(s => s.trim()).filter(s => s.length > 0);
    } else {
      filterRequest.skills = [];
    }

    console.log('Filter Request Being Sent:', filterRequest);
    this.filterCandidates(this.loginHrId, filterRequest);
  }

  filterCandidates(loginHrId: string, candidateFilterRequestDTO: CandidateFilterRequestDTO) {
    this.hrService.filterCandidates(loginHrId, candidateFilterRequestDTO).subscribe({
      next: (data) => {
        console.log('Filtered Candidates Response:', data);
        this.displayedCandidates = data;
      },
      error: (err) => {
        console.error("Exception occurred while calling filter API", err);
        this.showError('Error filtering candidates. Please check your filter criteria and try again.');
      },
      complete: () => {
        console.log("Filter completed successfully");
      }
    });
  }

  clearFilters() {
    this.selectedExperience = '';
    this.minSalary = null;
    this.maxSalary = null;
    this.selectedNoticePeriod = null;
    this.selectedSkills = '';
    this.displayedCandidates = this.allCandidates;
  }

  getInitials(fullName: string): string {
    if (!fullName) return '??';
    const names = fullName.split(' ');
    if (names.length >= 2) {
      return (names[0][0] + names[1][0]).toUpperCase();
    }
    return fullName.substring(0, 2).toUpperCase();
  }

  getSkillsArray(skills: string): string[] {
    if (!skills) return [];
    return skills.split(',').map(skill => skill.trim()).slice(0, 4);
  }

  formatDate(dateString: string): string {
    if (!dateString) return 'N/A';
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' });
  }

  changeStatusOfCandidateToShortListed(applicationId: number) {
    const candidate = this.displayedCandidates.find(c => c.applicationId === applicationId);
    if (candidate) {
      this.selectedApplicationId = applicationId;
      this.selectedCandidateName = candidate.fullName;
      this.confirmAction = 'shortlist';
      this.showConfirmModal = true;
    }
  }

  changeStatusOfCandidateToNotShortListed(applicationId: number) {
    const candidate = this.displayedCandidates.find(c => c.applicationId === applicationId);
    if (candidate) {
      this.selectedApplicationId = applicationId;
      this.selectedCandidateName = candidate.fullName;
      this.confirmAction = 'reject';
      this.showConfirmModal = true;
    }
  }

  confirmStatusChange() {
    this.showConfirmModal = false;

    if (this.selectedApplicationId === null || this.confirmAction === null) return;

    const status = this.confirmAction === 'shortlist' ? this.shortlistedStatus : this.notShortlistedStatus;

    this.hrService.changeStatusOfCandidateToShortListed(this.selectedApplicationId, status).subscribe({
      next: (data) => {
        console.log(data);
        this.showSuccess(data || `Candidate ${this.confirmAction === 'shortlist' ? 'shortlisted' : 'rejected'} successfully!`);
        this.refreshCandidates();
      },
      error: (err) => {
        console.error("Exception occurred while changing candidate status", err);
        const errorMsg = err.error?.message || err.error || `Failed to ${this.confirmAction} candidate. Please try again.`;
        this.showError(errorMsg);
      }
    });

    this.selectedApplicationId = null;
    this.selectedCandidateName = '';
    this.confirmAction = null;
  }

  cancelConfirmation() {
    this.showConfirmModal = false;
    this.selectedApplicationId = null;
    this.selectedCandidateName = '';
    this.confirmAction = null;
  }

  showSuccess(message: string) {
    this.successMessage = message;
    this.showSuccessModal = true;
  }

  showError(message: string) {
    this.errorMessage = message;
    this.showErrorModal = true;
  }

  closeSuccessModal() {
    this.showSuccessModal = false;
    this.successMessage = '';
  }

  closeErrorModal() {
    this.showErrorModal = false;
    this.errorMessage = '';
  }

  refreshCandidates() {
    this.getAllCandidates(this.loginHrId);
  }

  getConfirmationMessage(): string {
    if (this.confirmAction === 'shortlist') {
      return `Are you sure you want to shortlist ${this.selectedCandidateName}?`;
    } else if (this.confirmAction === 'reject') {
      return `Are you sure you want to reject ${this.selectedCandidateName}?`;
    }
    return '';
  }

  getConfirmationTitle(): string {
    if (this.confirmAction === 'shortlist') {
      return 'Confirm Shortlist';
    } else if (this.confirmAction === 'reject') {
      return 'Confirm Rejection';
    }
    return 'Confirm Action';
  }
}





Applied candidates html 
<!-- Add these modals at the end of applied-candidates.html, before closing </div> -->

<!-- Confirmation Modal -->
<div *ngIf="showConfirmModal" class="modal-overlay">
  <div class="modal-container confirmation-modal">
    <div class="modal-header">
      <div class="modal-icon" [ngClass]="confirmAction === 'shortlist' ? 'success-icon-header' : 'danger-icon-header'">
        <i [class]="confirmAction === 'shortlist' ? 'fas fa-user-check' : 'fas fa-user-times'"></i>
      </div>
      <h2>{{ getConfirmationTitle() }}</h2>
    </div>
    <div class="modal-body">
      <p>{{ getConfirmationMessage() }}</p>
      <div class="confirmation-details">
        <div class="detail-row">
          <span class="detail-label">Candidate:</span>
          <span class="detail-value">{{ selectedCandidateName }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Application ID:</span>
          <span class="detail-value">{{ selectedApplicationId }}</span>
        </div>
        <div class="detail-row">
          <span class="detail-label">Action:</span>
          <span class="detail-value" [ngClass]="confirmAction === 'shortlist' ? 'text-success' : 'text-danger'">
            {{ confirmAction === 'shortlist' ? 'Shortlist' : 'Reject' }}
          </span>
        </div>
      </div>
      <p class="confirmation-note" *ngIf="confirmAction === 'reject'">
        <i class="fas fa-exclamation-triangle"></i>
        This action will mark the candidate as rejected.
      </p>
      <p class="confirmation-note success-note" *ngIf="confirmAction === 'shortlist'">
        <i class="fas fa-info-circle"></i>
        This candidate will be moved to the shortlisted pool.
      </p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="cancelConfirmation()">
        <i class="fas fa-times"></i> Cancel
      </button>
      <button 
        type="button" 
        class="btn"
        [ngClass]="confirmAction === 'shortlist' ? 'btn-success' : 'btn-danger'"
        (click)="confirmStatusChange()"
      >
        <i [class]="confirmAction === 'shortlist' ? 'fas fa-check' : 'fas fa-ban'"></i>
        {{ confirmAction === 'shortlist' ? 'Shortlist' : 'Reject' }}
      </button>
    </div>
  </div>
</div>

<!-- Success Modal -->
<div *ngIf="showSuccessModal" class="modal-overlay">
  <div class="modal-container success-modal">
    <div class="modal-header success-header">
      <div class="success-icon">
        <i class="fas fa-check-circle"></i>
      </div>
      <h2>Success!</h2>
    </div>
    <div class="modal-body">
      <p>{{ successMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-success" (click)="closeSuccessModal()">
        <i class="fas fa-thumbs-up"></i> OK
      </button>
    </div>
  </div>
</div>

<!-- Error Modal -->
<div *ngIf="showErrorModal" class="modal-overlay">
  <div class="modal-container error-modal">
    <div class="modal-header error-header">
      <div class="error-icon">
        <i class="fas fa-exclamation-circle"></i>
      </div>
      <h2>Error</h2>
    </div>
    <div class="modal-body">
      <p>{{ errorMessage }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-danger" (click)="closeErrorModal()">
        <i class="fas fa-redo"></i> Close
      </button>
    </div>
  </div>
</div>






applied compenets CSS
/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.65);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  animation: fadeIn 0.2s ease-in;
  backdrop-filter: blur(4px);
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Modal Container */
.modal-container {
  background: white;
  border-radius: 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 520px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes slideUp {
  from {
    transform: translateY(40px) scale(0.95);
    opacity: 0;
  }
  to {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
}

/* Modal Header */
.modal-header {
  padding: 32px 28px 24px;
  border-bottom: 1px solid #e5e7eb;
  text-align: center;
}

.modal-header h2 {
  margin: 12px 0 0 0;
  font-size: 1.5rem;
  font-weight: 700;
  color: #1f2937;
}

/* Modal Icons */
.modal-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 72px;
  border-radius: 50%;
  font-size: 36px;
  margin-bottom: 8px;
  animation: iconBounce 0.6s ease-out;
}

@keyframes iconBounce {
  0% {
    transform: scale(0) rotate(0deg);
  }
  50% {
    transform: scale(1.15) rotate(180deg);
  }
  100% {
    transform: scale(1) rotate(360deg);
  }
}

.success-icon-header {
  background: linear-gradient(135deg, #d1fae5 0%, #a7f3d0 100%);
  color: #059669;
}

.danger-icon-header {
  background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
  color: #dc2626;
}

/* Success Header */
.success-header {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  border-bottom: none;
  padding: 36px 28px;
}

.success-header h2 {
  color: white;
}

.success-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 80px;
  height: 80px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 48px;
  margin-bottom: 12px;
  animation: successPulse 0.8s ease-out;
}

@keyframes successPulse {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
  100% {
    transform: scale(1);
  }
}

/* Error Header */
.error-header {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  border-bottom: none;
  padding: 36px 28px;
}

.error-header h2 {
  color: white;
}

.error-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 80px;
  height: 80px;
  background-color: rgba(255, 255, 255, 0.25);
  color: white;
  border-radius: 50%;
  font-size: 48px;
  margin-bottom: 12px;
  animation: errorShake 0.6s ease-in-out;
}

@keyframes errorShake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-12px) rotate(-5deg); }
  40% { transform: translateX(12px) rotate(5deg); }
  60% { transform: translateX(-8px) rotate(-3deg); }
  80% { transform: translateX(8px) rotate(3deg); }
}

/* Modal Body */
.modal-body {
  padding: 32px 28px;
}

.modal-body > p:first-child {
  margin: 0 0 24px 0;
  color: #374151;
  line-height: 1.6;
  font-size: 1.1rem;
  text-align: center;
  font-weight: 500;
}

.success-modal .modal-body p,
.error-modal .modal-body p {
  font-size: 1.15rem;
  color: #1f2937;
  font-weight: 600;
}

/* Confirmation Details */
.confirmation-details {
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  padding: 24px;
  margin: 24px 0;
  text-align: left;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 0;
  border-bottom: 1px solid #e5e7eb;
}

.detail-row:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

.detail-row:first-child {
  padding-top: 0;
}

.detail-label {
  font-weight: 600;
  color: #6b7280;
  font-size: 0.95rem;
}

.detail-value {
  color: #1f2937;
  font-weight: 700;
  font-size: 1rem;
}

.text-success {
  color: #059669 !important;
}

.text-danger {
  color: #dc2626 !important;
}

/* Confirmation Note */
.confirmation-note {
  margin-top: 24px;
  padding: 14px 18px;
  background-color: #fef3c7;
  border-left: 4px solid #f59e0b;
  border-radius: 6px;
  color: #92400e;
  font-size: 0.95rem;
  text-align: left;
  line-height: 1.5;
}

.confirmation-note.success-note {
  background-color: #d1fae5;
  border-left-color: #10b981;
  color: #065f46;
}

.confirmation-note i {
  margin-right: 8px;
  font-size: 1rem;
}

/* Modal Footer */
.modal-footer {
  padding: 24px 28px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: center;
  gap: 14px;
  background-color: #f9fafb;
  border-radius: 0 0 16px 16px;
}

/* Buttons */
.btn {
  padding: 13px 32px;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.25s ease;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  min-width: 130px;
  justify-content: center;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.btn i {
  font-size: 16px;
}

.btn-success {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
}

.btn-success:hover {
  background: linear-gradient(135deg, #059669 0%, #047857 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(16, 185, 129, 0.4);
}

.btn-danger {
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
}

.btn-danger:hover {
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(239, 68, 68, 0.4);
}

.btn-secondary {
  background-color: white;
  color: #4b5563;
  border: 2px solid #d1d5db;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.btn-secondary:hover {
  background-color: #f9fafb;
  border-color: #9ca3af;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.btn:active {
  transform: translateY(0);
}

/* Responsive Design */
@media (max-width: 600px) {
  .modal-container {
    width: 95%;
    margin: 20px;
    border-radius: 12px;
  }

  .modal-header,
  .modal-body,
  .modal-footer {
    padding: 24px 20px;
  }

  .modal-icon,
  .success-icon,
  .error-icon {
    width: 64px;
    height: 64px;
    font-size: 32px;
  }

  .modal-header h2 {
    font-size: 1.3rem;
  }

  .modal-body > p:first-child {
    font-size: 1rem;
  }

  .detail-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 6px;
  }

  .btn {
    width: 100%;
    min-width: unset;
    padding: 12px 24px;
  }

  .modal-footer {
    flex-direction: column-reverse;
    gap: 10px;
  }

  .confirmation-details {
    padding: 20px;
  }
}
