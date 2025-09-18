// your code goes here
document.addEventListener('DOMContentLoaded', function() {
    // Initialize data
    let feedbackData = JSON.parse(localStorage.getItem('feedbackData')) || [];
    
    // Initialize charts
    const sentimentCtx = document.getElementById('sentimentChart').getContext('2d');
    const categoryCtx = document.getElementById('categoryChart').getContext('2d');
    
    let sentimentChart = new Chart(sentimentCtx, {
        type: 'doughnut',
        data: {
            labels: ['Positive', 'Neutral', 'Negative'],
            datasets: [{
                data: [0, 0, 0],
                backgroundColor: [
                    'rgba(76, 175, 80, 0.7)',
                    'rgba(255, 152, 0, 0.7)',
                    'rgba(244, 67, 54, 0.7)'
                ],
                borderColor: [
                    'rgba(76, 175, 80, 1)',
                    'rgba(255, 152, 0, 1)',
                    'rgba(244, 67, 54, 1)'
                ],
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: {
                    position: 'bottom'
                }
            }
        }
    });
    
    let categoryChart = new Chart(categoryCtx, {
        type: 'bar',
        data: {
            labels: ['Product', 'Service', 'Support', 'Website', 'Other'],
            datasets: [{
                label: 'Number of Feedback',
                data: [0, 0, 0, 0, 0],
                backgroundColor: 'rgba(67, 97, 238, 0.7)',
                borderColor: 'rgba(67, 97, 238, 1)',
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: {
                    beginAtZero: true,
                    ticks: {
                        precision: 0
                    }
                }
            }
        }
    });
    
    // Form submission
    document.getElementById('feedbackForm').addEventListener('submit', function(e) {
        e.preventDefault();
        
        const customerName = document.getElementById('customerName').value;
        const customerEmail = document.getElementById('customerEmail').value;
        const category = document.getElementById('category').value;
        const rating = parseInt(document.getElementById('rating').value);
        const feedback = document.getElementById('feedback').value;
        
        // Determine sentiment based on rating
        let sentiment;
        if (rating >= 4) {
            sentiment = 'positive';
        } else if (rating === 3) {
            sentiment = 'neutral';
        } else {
            sentiment = 'negative';
        }
        
        // Create feedback object
        const newFeedback = {
            id: Date.now(),
            customerName,
            customerEmail,
            category,
            rating,
            feedback,
            sentiment,
            date: new Date().toISOString()
        };
        
        // Add to data array
        feedbackData.push(newFeedback);
        
        // Save to localStorage
        localStorage.setItem('feedbackData', JSON.stringify(feedbackData));
        
        // Reset form
        this.reset();
        
        // Show notification
        showNotification('Feedback added successfully!', 'success');
        
        // Update UI
        updateStats();
        updateCharts();
        updateKeywords();
        renderFeedbackList();
    });
    
    // Filter and sort controls
    document.getElementById('filterCategory').addEventListener('change', renderFeedbackList);
    document.getElementById('filterSentiment').addEventListener('change', renderFeedbackList);
    document.getElementById('sortBy').addEventListener('change', renderFeedbackList);
    
    // Functions
    function updateStats() {
        const totalFeedback = feedbackData.length;
        const positiveCount = feedbackData.filter(item => item.sentiment === 'positive').length;
        const neutralCount = feedbackData.filter(item => item.sentiment === 'neutral').length;
        const negativeCount = feedbackData.filter(item => item.sentiment === 'negative').length;
        
        document.getElementById('totalFeedback').textContent = totalFeedback;
        document.getElementById('positiveCount').textContent = positiveCount;
        document.getElementById('neutralCount').textContent = neutralCount;
        document.getElementById('negativeCount').textContent = negativeCount;
    }
    
    function updateCharts() {
        // Update sentiment chart
        const positiveCount = feedbackData.filter(item => item.sentiment === 'positive').length;
        const neutralCount = feedbackData.filter(item => item.sentiment === 'neutral').length;
        const negativeCount = feedbackData.filter(item => item.sentiment === 'negative').length;
        
        sentimentChart.data.datasets[0].data = [positiveCount, neutralCount, negativeCount];
        sentimentChart.update();
        
        // Update category chart
        const categories = ['Product', 'Service', 'Support', 'Website', 'Other'];
        const categoryCounts = categories.map(category => 
            feedbackData.filter(item => item.category === category).length
        );
        
        categoryChart.data.datasets[0].data = categoryCounts;
        categoryChart.update();
    }
    
    function updateKeywords() {
        const keywordsContainer = document.getElementById('keywordsContainer');
        
        if (feedbackData.length === 0) {
            keywordsContainer.innerHTML = `
                <div class="empty-state">
                    <i class="fas fa-tag"></i>
                    <p>No keywords found. Add some feedback to see keywords.</p>
                </div>
            `;
            return;
        }
        
        // Simple keyword extraction (in a real app, you'd use more sophisticated NLP)
        const allFeedback = feedbackData.map(item => item.feedback.toLowerCase()).join(' ');
        const words = allFeedback.replace(/[^\w\s]/g, '').split(/\s+/);
        
        // Filter out common words
        const stopWords = ['the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should', 'may', 'might', 'must', 'can', 'this', 'that', 'these', 'those', 'i', 'you', 'he', 'she', 'it', 'we', 'they', 'me', 'him', 'her', 'us', 'them'];
        
        const filteredWords = words.filter(word => 
            word.length > 3 && !stopWords.includes(word)
        );
        
        // Count word frequency
        const wordCounts = {};
        filteredWords.forEach(word => {
            wordCounts[word] = (wordCounts[word] || 0) + 1;
        });
        
        // Sort by frequency and get top 10
        const sortedWords = Object.entries(wordCounts)
            .sort((a, b) => b[1] - a[1])
            .slice(0, 10);
        
        // Create keyword elements
        keywordsContainer.innerHTML = '';
        sortedWords.forEach(([word, count]) => {
            const keywordElement = document.createElement('span');
            keywordElement.className = 'keyword';
            keywordElement.textContent = word;
            keywordsContainer.appendChild(keywordElement);
        });
    }
    
    function renderFeedbackList() {
        const feedbackList = document.getElementById('feedbackList');
        const filterCategory = document.getElementById('filterCategory').value;
        const filterSentiment = document.getElementById('filterSentiment').value;
        const sortBy = document.getElementById('sortBy').value;
        
        // Filter feedback
        let filteredFeedback = feedbackData;
        
        if (filterCategory) {
            filteredFeedback = filteredFeedback.filter(item => item.category === filterCategory);
        }
        
        if (filterSentiment) {
            filteredFeedback = filteredFeedback.filter(item => item.sentiment === filterSentiment);
        }
        
        // Sort feedback
        filteredFeedback.sort((a, b) => {
            switch (sortBy) {
                case 'newest':
                    return new Date(b.date) - new Date(a.date);
                case 'oldest':
                    return new Date(a.date) - new Date(b.date);
                case 'highest':
                    return b.rating - a.rating;
                case 'lowest':
                    return a.rating - b.rating;
                default:
                    return 0;
            }
        });
        
        // Render feedback list
        if (filteredFeedback.length === 0) {
            feedbackList.innerHTML = `
                <div class="empty-state">
                    <i class="fas fa-comments"></i>
                    <p>No feedback matches your filters.</p>
                </div>
            `;
            return;
        }
        
        feedbackList.innerHTML = '';
        filteredFeedback.forEach(item => {
            const feedbackItem = document.createElement('div');
            feedbackItem.className = 'feedback-item';
            
            const date = new Date(item.date).toLocaleDateString();
            
            feedbackItem.innerHTML = `
                <div class="feedback-header">
                    <div>
                        <strong>${item.customerName}</strong>
                        <span class="sentiment sentiment-${item.sentiment}">${item.sentiment}</span>
                    </div>
                    <div>${'★'.repeat(item.rating)}${'☆'.repeat(5 - item.rating)}</div>
                </div>
                <div class="feedback-meta">
                    <span><i class="fas fa-envelope"></i> ${item.customerEmail}</span>
                    <span><i class="fas fa-folder"></i> ${item.category}</span>
                    <span><i class="fas fa-calendar"></i> ${date}</span>
                </div>
                <div class="feedback-content">
                    ${item.feedback}
                </div>
            `;
            
            feedbackList.appendChild(feedbackItem);
        });
    }
    
    function showNotification(message, type = 'success') {
        const notification = document.getElementById('notification');
        const notificationMessage = document.getElementById('notificationMessage');
        const icon = notification.querySelector('i');
        
        notification.className = `notification ${type}`;
        notificationMessage.textContent = message;
        
        if (type === 'success') {
            icon.className = 'fas fa-check-circle';
        } else if (type === 'error') {
            icon.className = 'fas fa-exclamation-circle';
        }
        
        notification.classList.add('show');
        
        setTimeout(() => {
            notification.classList.remove('show');
        }, 3000);
    }
    
    // Initialize UI
    updateStats();
    updateCharts();
    updateKeywords();
    renderFeedbackList();
});