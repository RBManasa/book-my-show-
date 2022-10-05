# Auto detect text files and perform LF normalization
* text=auto
from django.urls import path,include,re_path
from main import views
app_name = 'main'
urlpatterns = [
 path('', views.index, name='index'),
 path('signup/',views.user_signup,name="signup"),
 path('movies/', views.movies.as_view(), name='movies'),
 path('search/', views.search, name='search'),
 
re_path(r'^accounts/(?P<pk>\d+)/(?P<user>\w+)/',views.account.as_view(),name="
accounts"),
 re_path(r'^accountsupdate/(?P<pk>\d+)/',views.account_update.as_view(),name="accounts-update"),
 re_path(r'^accountsdelete/(?P<pk>\d+)/',views.account_delete.as_view(),name="accounts-delete"),
 re_path(r'^moviedetail/(?P<pk>\d+)/',views.movie_detail_view.as_view(),name="movie-detailview"),
 re_path(r'^theatrelist/(?P<pk>\d+)/',views.movie_booking_theatre.as_view(),name="movie-bookingtheatre"),
 re_path(r'^booking/(?P<pk>\d+)/',views.show_details.as_view(),name="moviebooking"),
 re_path(r'^booking/(?P<pk>\d+)/',views.booking,name="booking"),
 re_path(r'^bookticket/(?P<pk>\d+)/(?P<id>\d+)',views.book_ticket,name="book-ticket"),
 re_path(r'^booked/(?P<pk>\d+)/',views.booked.as_view(),name="booked"),
 re_path(r'^cancel/(?P<pk>\d+)/',views.cancel_ticket,name="cancel_ticket"),
]
from django.shortcuts import render,redirect
from django.http import 
HttpResponse,HttpResponseForbidden,HttpResponseRedirect,HttpResponseBadRe
quest
from dashboard import models
from dashboard.models import movie,show,theatre,screen,User
from django.views.generic import 
DetailView,ListView,TemplateView,UpdateView,DeleteView
from django.db import connection
from django.contrib.auth.forms import UserCreationForm
from django.urls import reverse_lazy
from django.contrib.auth.mixins import 
LoginRequiredMixin,PermissionRequiredMixin
from django.db.models import Q
from django.contrib import messages
from main.models import TicketsForm,booking
from datetime import datetime, timedelta
from django.contrib.auth.decorators import login_required
# Create your views here.
def index(request) :
 slider_images = movie.objects.raw('SELECT id,slideshow_image FROM 
dashboard_movie ORDER BY id DESC LIMIT 4')
 thumbnail_images = movie.objects.raw('SELECT id,thumbnail_image FROM 
dashboard_movie ORDER BY id DESC LIMIT 8')
 trailer = movie.objects.raw('SELECT id,trailer FROM dashboard_movie 
ORDER BY id DESC LIMIT 6')
 return render(request,'main/index.html',{ 
'slider_images':slider_images,'thumbnail_images' : thumbnail_images,'trailer' : 
trailer })
class movies(ListView) :
 context_object_name = 'movie'
 model = models.movie
 template_name = 'main/movie-view.html'
class movie_detail_view(DetailView) :
 context_object_name = 'movie'
 model = models.movie
 template_name = 'main/movie-detail-view.html'
class movie_booking_theatre(DetailView) :
 context_object_name = 'movie'
 model = models.movie
 template_name = 'main/movie-booking-theatre.html'
class show_details(LoginRequiredMixin,DetailView) :
 def get_context_data(self, **kwargs):
 context = super().get_context_data(**kwargs) 
 context['form'] = TicketsForm
 show_id = self.kwargs['pk']
 Show = show.objects.get(pk=show_id)
 context['tickets_booked'] = booking.objects.filter(show=Show)
 return context
 context_object_name = 'show'
 model = models.show
 template_name = 'main/booking-page.html'
@login_required
def book_ticket(request,pk=None,id=None):
 
 data = TicketsForm(request.POST)
 Booking = data.save(commit=False) 
 Show = show.objects.get(pk=pk)
 user = User.objects.get(pk=id)
 Booking.show = Show
 Booking.user = user
 if Booking.row_num == 0 or Booking.col_num == 0 or Booking.row_num > 
Show.screen.no_of_rows or Booking.col_num > Show.screen.no_of_columns:
 return HttpResponseBadRequest("Invalid ticket number")
 if not request.session.session_key:
 request.session.save()
 Booking.session = request.session.session_key 
 try:
 existing_tix = booking.objects.get(row_num=Booking.row_num, 
col_num=Booking.col_num, show=Show)
 if existing_tix.session == Booking.session and existing_tix.status == 1:
 if existing_tix.status == 1:
 existing_tix.delete()
 else:
 Booking = existing_tix
 else:
 return HttpResponseBadRequest("This ticket has already been reserved")
 except booking.DoesNotExist:
 pass
 Booking.save()
 return redirect('main:booked',pk=pk)
class account(LoginRequiredMixin,DetailView) :
 def get_context_data(self, **kwargs):
 context = super().get_context_data(**kwargs)
 user_id = self.kwargs['pk']
 username = self.kwargs['user']
 u = User.objects.get(pk=user_id)
 context['tickets_booked'] = booking.objects.filter(user=u)
 if u.username == username :
 context['test'] = username
 return context
 context_object_name = 'user'
 model = models.User
 template_name = 'main/account.html'
class account_update(LoginRequiredMixin,UpdateView) :
 context_object_name = 'user'
 model = models.User
 fields = ['username','first_name','last_name','email',]
 template_name = 'main/account_form.html'
 success_url = reverse_lazy('main:index')
class account_delete(LoginRequiredMixin,DeleteView) :
 context_object_name = 'user'
 model = models.User
 template_name = 'main/user_confirm_delete.html'
 success_url = reverse_lazy('main:index')
def user_signup(request) :
 if request.method == 'POST' :
 form = UserCreationForm(request.POST)
 if form.is_valid() : 
 user = form.save()
 return redirect('main:index')
 else :
 form = UserCreationForm()
 return render(request,'main/signup.html',{'form':form})
def search(request) :
 if request.method == 'POST' :
 search_query = request.POST['search']
 if search_query :
 result = movie.objects.filter(Q(name_icontains=search_query) | 
Q(heroicontains=search_query) | Q(heroine_icontains=search_query))
 if result :
 return render(request,'main/search.html',{'results':result,'search_query' : 
search_query})
 else :
 messages.error(request,'Sorry no results found')
 else :
 return HttpResponseRedirect('/')
 return render(request,'main/search.html')
class booked(LoginRequiredMixin,DetailView) :
 context_object_name = 'show'
 model = models.show
 template_name = 'main/confirm_movie.html'
def cancel_ticket(request,pk=None) :
 Ticket = booking.objects.get(pk=pk)
 Ticket.delete()
 return redirect('main:index')
from django.db import models
from django.forms import ModelForm
from dashboard.models import show
from django.contrib.auth.models import User
# Create your models here.
TICKET_STATUS_CHOICES = (
 (1, 'AVAILABLE'),
 (2, 'BLOCKED'),
 (3, 'BOOKED')
)
class booking(models.Model):
 name = models.CharField(max_length=50)
 age = models.PositiveSmallIntegerField(null=False)
 created_at = models.DateTimeField(auto_now_add=True, db_index=True)
 updated_at = models.DateTimeField(auto_now=True)
 row_num = models.PositiveSmallIntegerField(null=False, blank=False)
 col_num = models.PositiveSmallIntegerField(null=False, blank=False)
 show = 
models.ForeignKey(show,related_name="booking_show",null=False,on_delete=m
odels.CASCADE)
 user = 
models.ForeignKey(User,related_name="booking_user",null=False,default=None, 
on_delete=models.CASCADE)
 status = models.IntegerField(choices=TICKET_STATUS_CHOICES, default=1)
 session = models.CharField(blank=False, null=False, max_length=200)
 class Meta:
 unique_together = ('show', 'row_num', 'col_num')
class TicketsForm(ModelForm):
 class Meta:
 model = booking
 fields = ['name','age','row_num', 'col_num',]
from django.apps import AppConfig
class MainConfig(AppConfig):
 name = 'main'