// script.js
document.addEventListener('DOMContentLoaded', function() {
    // Signature functionality
    const signatureCanvas = document.getElementById('signatureCanvas');
    const witnessCanvas = document.getElementById('witnessSignatureCanvas');
    const finalCanvas = document.getElementById('finalSignatureCanvas');
    
    const signatureInput = document.getElementById('applicantSignature');
    const witnessInput = document.getElementById('witnessSignature');
    const finalInput = document.getElementById('finalSignature');
    
    // Initialize signature pads
    const signaturePad = new SignaturePad(signatureCanvas);
    const witnessPad = new SignaturePad(witnessCanvas);
    const finalPad = new SignaturePad(finalCanvas);
    
    // Clear buttons
    document.getElementById('clearSignature').addEventListener('click', function() {
        signaturePad.clear();
        signatureInput.value = '';
    });
    
    document.getElementById('clearWitnessSignature').addEventListener('click', function() {
        witnessPad.clear();
        witnessInput.value = '';
    });
    
    document.getElementById('clearFinalSignature').addEventListener('click', function() {
        finalPad.clear();
        finalInput.value = '';
    });
    
    // Save signatures when form is submitted
    document.getElementById('loanApplicationForm').addEventListener('submit', function(e) {
        if (signaturePad.isEmpty()) {
            alert('Please provide your signature');
            e.preventDefault();
            return;
        }
        
        if (witnessPad.isEmpty()) {
            alert('Please provide witness signature');
            e.preventDefault();
            return;
        }
        
        if (finalPad.isEmpty()) {
            alert('Please sign the terms and conditions');
            e.preventDefault();
            return;
        }
        
        // Convert signatures to data URLs and store in hidden inputs
        signatureInput.value = signaturePad.toDataURL();
        witnessInput.value = witnessPad.toDataURL();
        finalInput.value = finalPad.toDataURL();
    });
    
    // Show/hide other purpose field
    const loanPurposeRadios = document.querySelectorAll('input[name="loanPurpose"]');
    const otherPurposeField = document.getElementById('otherPurpose');
    
    loanPurposeRadios.forEach(radio => {
        radio.addEventListener('change', function() {
            if (this.value === 'other') {
                otherPurposeField.style.display = 'block';
                otherPurposeField.required = true;
            } else {
                otherPurposeField.style.display = 'none';
                otherPurposeField.required = false;
                otherPurposeField.value = '';
            }
        });
    });
    
    // Calculate loan details
    const loanAmountInput = document.getElementById('loanAmount');
    const deductionInput = document.getElementById('deduction');
    const fortnightsInput = document.getElementById('fortnights');
    const grossLoanInput = document.getElementById('grossLoan');
    
    const grossSalaryInput = document.getElementById('grossSalary');
    const loanDeductionsInput = document.getElementById('loanDeductions');
    const totalDeductionsInput = document.getElementById('totalDeductions');
    const netSalaryInput = document.getElementById('netSalary');
    
    // Auto-calculate gross loan
    loanAmountInput.addEventListener('input', calculateGrossLoan);
    deductionInput.addEventListener('input', calculateGrossLoan);
    fortnightsInput.addEventListener('input', calculateGrossLoan);
    
    function calculateGrossLoan() {
        const amount = parseFloat(loanAmountInput.value) || 0;
        const deduction = parseFloat(deductionInput.value) || 0;
        const fortnights = parseInt(fortnightsInput.value) || 0;
        
        const grossLoan = amount + (deduction * fortnights);
        grossLoanInput.value = grossLoan.toFixed(2);
    }
    
    // Auto-calculate net salary
    grossSalaryInput.addEventListener('input', calculateNetSalary);
    loanDeductionsInput.addEventListener('input', calculateNetSalary);
    totalDeductionsInput.addEventListener('input', calculateNetSalary);
    
    function calculateNetSalary() {
        const grossSalary = parseFloat(grossSalaryInput.value) || 0;
        const loanDeductions = parseFloat(loanDeductionsInput.value) || 0;
        const totalDeductions = parseFloat(totalDeductionsInput.value) || 0;
        
        const netSalary = grossSalary - loanDeductions - totalDeductions;
        netSalaryInput.value = netSalary.toFixed(2);
    }
    
    // Save as draft functionality
    document.getElementById('saveDraft').addEventListener('click', function() {
        // In a real application, you would save the form data to local storage or a server
        alert('Draft saved locally (implementation would save to storage)');
    });
    
    // Form validation
    document.getElementById('loanApplicationForm').addEventListener('submit', function(e) {
        // Check required fields
        const requiredFields = document.querySelectorAll('[required]');
        let isValid = true;
        
        requiredFields.forEach(field => {
            if (!field.value) {
                field.style.borderColor = 'red';
                isValid = false;
            } else {
                field.style.borderColor = '#ddd';
            }
        });
        
        if (!isValid) {
            e.preventDefault();
            alert('Please fill in all required fields');
        }
    });
    
    // Signature Pad implementation (simplified)
    function SignaturePad(canvas) {
        let ctx = canvas.getContext('2d');
        let drawing = false;
        let lastX = 0;
        let lastY = 0;
        
        // Set up canvas
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 2;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        
        // Event listeners
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mouseout', stopDrawing);
        
        // Touch support
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchmove', handleTouchMove);
        canvas.addEventListener('touchend', handleTouchEnd);
        
        function startDrawing(e) {
            drawing = true;
            const pos = getPosition(e);
            lastX = pos.x;
            lastY = pos.y;
        }
        
        function draw(e) {
            if (!drawing) return;
            
            const pos = getPosition(e);
            
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(pos.x, pos.y);
            ctx.stroke();
            
            lastX = pos.x;
            lastY = pos.y;
        }
        
        function stopDrawing() {
            drawing = false;
        }
        
        function handleTouchStart(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const mouseEvent = new MouseEvent('mousedown', {
                clientX: touch.clientX,
                clientY: touch.clientY
            });
            canvas.dispatchEvent(mouseEvent);
        }
        
        function handleTouchMove(e) {
            e.preventDefault();
            const touch =
