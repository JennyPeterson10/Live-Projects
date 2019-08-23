# Live Projects

After the courses at The Tech Academy, I worked with a team of developers on various web applications projects for clients. Below are samples of the stories I worked on for these projects.

## Python

As part of a larger project to develop a travel application, I created a calendar that allows users to add events and activities for their travel plans.

    # Event class
    class Event(models.Model):
        title = models.CharField('Event Title', max_length=200)
        start= models.DateTimeField('Start Date/Time')
        end = models.DateTimeField('End Date/Time', blank=True, null=True)
        notes = models.TextField(blank=True, null=True)

        @property
        def get_html_url(self):
            url = reverse('event_edit', args=(self.id,))
            return f'<a href="{url}"> {self.title} </a>'
            
    # Views		
    def event(request, event_id=None):
        instance = Event()
        if event_id:
            instance = get_object_or_404(Event, pk=event_id)
        else:
            instance = Event()

        form = EventForm(request.POST or None, instance=instance)
        if request.POST and form.is_valid():
            form.save()
            return HttpResponseRedirect(reverse('calendar'))
        return render(request, 'CalendarApp/event.html', {'form': form})

    class CalendarView(generic.ListView):
        model = Event
        template_name = 'CalendarApp/calendar.html'

        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)

            # Use today's date for the calendar month
            d = get_date(self.request.GET.get('month', None))
            cal = Calendar(d.year, d.month)

            # Call the formatmonth method, which returns our calendar as a table
            html_cal = cal.formatmonth(withyear=True)
            context['calendar'] = mark_safe(html_cal)
            context['prev_month'] = prev_month(d)
            context['next_month'] = next_month(d)
            return context

    def get_date(req_month):
        if req_month:
            year, month = (int(x) for x in req_month.split('-'))
            return date(year, month, day=1)
        return datetime.today()

    def prev_month(d):
        first = d.replace(day=1)
        prev_month = first - timedelta(days=1)
        month = 'month=' + str(prev_month.year) + '-' + str(prev_month.month)
        return month

    def next_month(d):
        days_in_month = calendar.monthrange(d.year, d.month)[1]
        last = d.replace(day=days_in_month)
        next_month = last + timedelta(days=1)
        month = 'month=' + str(next_month.year) + '-' + str(next_month.month)
        return month

## C#

I worked on a request to generate a confirmation code for a new user. When a new user requests access to the management portal, an admin gives them a user name and the generated confirmation code. 

        // POST: CreateUserRequest/Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "UserCreationRequestId,UserName,ConfirmationCode")] CreateUserRequest createUserRequest)
        {
            if (ModelState.IsValid)
            {
                createUserRequest.UserCreationRequestId = Guid.NewGuid();
                createUserRequest.ConfirmationCode = Random(100000,1000000);
                db.CreateUserRequests.Add(createUserRequest);
                db.SaveChanges();
                return RedirectToAction("ConfirmationCode", new { id = createUserRequest.UserCreationRequestId });
            }
            return View(createUserRequest);
        }

        // Creates a ramdomly generated code for the confirmation code
        private int Random(int min, int max)
        {
            Random random = new Random();
            return random.Next(min, max);
        }

        // GET: CreateUserRequest/ConfirmationCode
        public ActionResult ConfirmationCode(Guid? id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            CreateUserRequest createUserRequest = db.CreateUserRequests.Find(id);
            if (createUserRequest == null)
            {
                return HttpNotFound();
            }
            return View(createUserRequest);
        }
