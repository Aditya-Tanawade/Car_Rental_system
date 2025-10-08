2025-10-08T17:19:16.246+05:30  WARN 20832 --- [hrportalnew] [nio-8081-exec-8] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [org.springframework.web.HttpMediaTypeNotSupportedException: Content-Type 'text/plain;charset=UTF-8' is not supported]


package com.finalproject.hrportal.dto;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class SaveInterviewDTO {
    private String candidateEmail;
    private String employeeEmail;
    @JsonDeserialize
    private String meetingLink;
    private String interviewDate;
    private String startTime;
    private String endTime;
    private String summary;
    private String description;
}





 @PatchMapping("/update-interview/details/{applicationId}")
    public ResponseEntity<String>updateInterviewDetails(@RequestBody SaveInterviewDTO saveInterviewDTO,@PathVariable ("applicationId") int  applicationId){
        return ResponseEntity.ok(hrService.updateInterviewDetails(saveInterviewDTO,applicationId));

    }


	
  scheduleInterview(InterviewRequest:InterviewRequest):Observable<string>{
    return this.httpClient.post("http://localhost:8081/interviews/schedule",InterviewRequest,{responseType:'text'});
  }


  saveMeetingLinkAndNotify(saveInterviewDTO:SaveInterviewDTO,applicationId:number):Observable<string>{
        return this.httpClient.patch(this.baseurl+"update-interview/details/"+applicationId,{InterviewRequest},{responseType:'text'});
  }


export class SaveInterviewDTO{
    candidateEmail: string="";
    employeeEmail: string="";
      meetingLink: string="";
      interviewDate:string="";
      startTime:string="";
      endTime:string="";
      summary:string="";
      description:string="";
}



// Submit schedule request
  submitScheduleInterview() {
    if (!this.validateForm()) {
      return;
    }

    this.scheduleError = '';

    const requestBody: InterviewRequest = {
      candidateEmail: this.interviewForm.candidateEmail,
      employeeEmail: this.interviewForm.employeeEmail,
      startTime: this.convertToISO8601(this.interviewForm.interviewDate, this.interviewForm.startTime),
      endTime: this.convertToISO8601(this.interviewForm.interviewDate, this.interviewForm.endTime),
      summary: this.interviewForm.summary,
      description: this.interviewForm.description
    };

    this.saveInterviewDTO.candidateEmail=this.interviewForm.candidateEmail;
    this.saveInterviewDTO.employeeEmail=this.interviewForm.employeeEmail;
    this.saveInterviewDTO.startTime=this.interviewForm.startTime
    this.saveInterviewDTO.interviewDate=this.interviewForm.interviewDate
    this.saveInterviewDTO.endTime=this.interviewForm.endTime
    this.saveInterviewDTO.summary=this.interviewForm.summary,
    this.saveInterviewDTO.description=this.interviewForm.description,
   

    console.log(this.saveInterviewDTO);
    

    console.log('Sending interview request:', requestBody);

    if(this.selectedCandidate!=null){
    this.applicationId=this.selectedCandidate.applicationId;
    }
    console.log(this.applicationId);
    
    this.hrService.scheduleInterview(requestBody).subscribe({
      next: (response: any) => {
        console.log('Interview scheduled successfully:', response);
        this.zoomMeetingLink = response;
        this.saveInterviewDTO.meetingLink=response;
        this.showMeetingLink = true;
        this.getAllShortlistedCandidates(this.loginHrId);
      },
      error: (err) => {
        console.error('Error scheduling interview:', err);
        this.scheduleError = err.error?.message || 'Failed to schedule interview. Please try again.';
      }
    });
  }

  // Save meeting link to database and notify
  notifyCandidate() {
    if (!this.zoomMeetingLink || !this.selectedCandidate) {
      return;
    }
    console.log(this.saveInterviewDTO);
    
    this.hrService.saveMeetingLinkAndNotify(this.saveInterviewDTO,this.applicationId).subscribe({
      next: (response) => {

        console.log('Notification sent successfully:', response);
        console.log(this.saveInterviewDTO);
        alert('Meeting link saved and candidate notified successfully!');
        this.closeScheduleModal();
        this.getAllShortlistedCandidates(this.loginHrId);
      },
      error: (err) => {
        console.error('Error sending notification:', err);
        alert('Failed to send notification. Please try again.');
      }
    });
  }



<!-- No Results Message -->
  <div *ngIf="filterShortlistedCandidates.length === 0" class="no-results">
    <i class="fas fa-inbox fa-3x"></i>
    <p>No candidates found matching your criteria.</p>
  </div>
</div>

<!-- Schedule Interview Modal -->
<div class="modal-overlay" *ngIf="showScheduleModal" (click)="closeScheduleModal()">
  <div class="modal-container" (click)="$event.stopPropagation()">
    <div class="modal-header">
      <h2 class="modal-title">
        <i class="fas fa-calendar-alt"></i>
        Schedule {{ interviewRound }}
      </h2>
      <button class="modal-close" (click)="closeScheduleModal()">
        <i class="fas fa-times"></i>
      </button>
    </div>

    <div class="modal-body">
      <!-- Candidate Info Banner -->
      <div class="candidate-info-banner" *ngIf="selectedCandidate">
        <div class="banner-avatar">{{ getInitials(selectedCandidate.fullName) }}</div>
        <div class="banner-details">
          <h3>{{ selectedCandidate.fullName }}</h3>
          <p>{{ selectedCandidate.title }} • {{ selectedCandidate.totalExperience }} years exp</p>
        </div>
      </div>

      <!-- Form Section -->
      <form class="interview-form" *ngIf="!showMeetingLink">
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">
              <i class="fas fa-user"></i> Candidate Email
            </label>
            <input 
              type="email" 
              class="form-input" 
              [(ngModel)]="interviewForm.candidateEmail"
              name="candidateEmail"
              readonly
            />
          </div>

          <div class="form-group">
            <label class="form-label">
              <i class="fas fa-user-tie"></i> Interviewer Email
            </label>
            <input 
              type="email" 
              class="form-input" 
              [(ngModel)]="interviewForm.employeeEmail"
              name="employeeEmail"
              placeholder="interviewer@company.com"
              required
            />
          </div>
        </div>

        <div class="form-row">
          <div class="form-group">
            <label class="form-label">
              <i class="fas fa-calendar"></i> Interview Date
            </label>
            <input 
              type="date" 
              class="form-input" 
              [(ngModel)]="interviewForm.interviewDate"
              name="interviewDate"
              [min]="getTomorrowDate()"
              required
            />
          </div>

          <div class="form-group">
            <label class="form-label">
              <i class="fas fa-clock"></i> Start Time
            </label>
            <input 
              type="time" 
              class="form-input" 
              [(ngModel)]="interviewForm.startTime"
              name="startTime"
              required
            />
          </div>

          <div class="form-group">
            <label class="form-label">
              <i class="fas fa-clock"></i> End Time
            </label>
            <input 
              type="time" 
              class="form-input" 
              [(ngModel)]="interviewForm.endTime"
              name="endTime"
              required
            />
          </div>
        </div>

        <div class="form-group">
          <label class="form-label">
            <i class="fas fa-heading"></i> Summary
          </label>
          <input 
            type="text" 
            class="form-input" 
            [(ngModel)]="interviewForm.summary"
            name="summary"
            placeholder="e.g., Technical Interview - Backend Developer"
            required
          />
        </div>

        <div class="form-group">
          <label class="form-label">
            <i class="fas fa-align-left"></i> Description
          </label>
          <textarea 
            class="form-textarea" 
            [(ngModel)]="interviewForm.description"
            name="description"
            rows="4"
            placeholder="Enter interview description and topics to be discussed..."
            required
          ></textarea>
        </div>

        <div class="error-message" *ngIf="scheduleError">
          <i class="fas fa-exclamation-circle"></i>
          {{ scheduleError }}
        </div>
      </form>

      <!-- Meeting Link Display -->
      <div class="meeting-link-container" *ngIf="showMeetingLink">
        <div class="success-icon">
          <i class="fas fa-check-circle"></i>
        </div>
        <h3 class="success-title">Interview Scheduled Successfully!</h3>
        <p class="success-message">The Zoom meeting has been created. Share this link with the candidate.</p>
        
        <div class="meeting-link-box">
          <label class="meeting-link-label">
            <i class="fas fa-video"></i> Zoom Meeting Link
          </label>
          <div class="meeting-link-display">
            <input 
              type="text" 
              class="meeting-link-input" 
              [value]="zoomMeetingLink" 
              readonly
            />
            <button 
              class="copy-btn" 
              (click)="copyToClipboard(zoomMeetingLink)"
              title="Copy to clipboard"
            >
              <i class="fas fa-copy"></i>
            </button>
          </div>
        </div>

        <div class="meeting-details">
          <div class="detail-item">
            <i class="fas fa-calendar-alt"></i>
            <span>{{ interviewForm.interviewDate | date: 'fullDate' }}</span>
          </div>
          <div class="detail-item">
            <i class="fas fa-clock"></i>
            <span>{{ interviewForm.startTime }} - {{ interviewForm.endTime }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="modal-footer">
      <button 
        class="btn btn-secondary" 
        (click)="closeScheduleModal()"
      >
        <i class="fas fa-times"></i> Cancel
      </button>

      <button 
        class="btn btn-success" 
        (click)="submitScheduleInterview()"
        *ngIf="!showMeetingLink"
      >
        <i class="fas fa-calendar-check"></i>
        Schedule Interview
      </button>

      <button 
        class="btn btn-primary" 
        (click)="notifyCandidate()"
        *ngIf="showMeetingLink"
      >
        <i class="fas fa-paper-plane"></i> Save 
      </button>
    </div>
  </div>


  
