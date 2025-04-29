
# CRUD Template

## cdn
```bash
https://cdnjs.cloudflare.com/ajax/libs/axios/1.9.0/axios.min.js
```
```bash
https://cdn.jsdelivr.net/npm/sweetalert2@11
```
```bash
https://cdn.datatables.net/1.13.11/css/dataTables.bootstrap5.min.css
https://cdn.datatables.net/1.13.11/js/jquery.dataTables.min.js
https://cdn.datatables.net/1.13.11/js/dataTables.bootstrap5.min.js
```

### ajax setup
```bash
		$.ajaxSetup({
			headers: {
				'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
			}
		})
```
### Initialize Datatable
```bash
		let table = $('#dataID').DataTable({
			processing: true,
			serverSide: true,
			order: [[4, 'desc']],
			ajax: {
				url: "{{ route('route.name.laravel') }}",
				type: 'POST'
			},
			columns: [
				{ data: 'name', name: 'name' },
            ],
            language: {
				paginate: {
					previous: "<i class='bi bi-chevron-left'>",
					next: "<i class='bi bi-chevron-right'>",
				},
			},
		})
```
### TOAST Function
```bash
		function showToast(type, message) {
			$.toast({
				heading: type === 'success' ? 'Sukses' : 'Error',
				text: message,
				icon: type,
				position: 'top-right',
				showHideTransition: 'slide',
				hideAfter: 3000
			})
		}
```
### Initialize select2
```bash
        function initializeSelect2(selector, url, placeholder = 'Pilih Data') {
			if ($(selector).hasClass('select2-hidden-accessible')) {
				$(selector).empty();
			} else {
				$(selector).select2({
					theme: 'bootstrap-5',
					placeholder: placeholder,
					allowClear: true,
					dropdownParent: $('#addDataModal'),
					ajax: {
						url: url,
						dataType: 'json',
						delay: 250,
						data: function (params) {
							return { q: params.term };
						},
						processResults: function (data) {
							return { results: data.results };
						},
						cache: true
					},
					minimumInputLength: 0
				})
			}
		}
```
### show modal add
```bash
        $('#btn_add').on('click', function () {
			$('#addDataForm')[0].reset()
			$('.invalid-feedback').hide()
			$('.form-control').removeClass('is-invalid')
	
			$('#addDataModalLabel').text('Tambah User')
			$('#addDataForm').removeData('id')
	
			initializeSelect2('#level', "{{ route('admin.levels') }}", 'Pilih Level')
			initializeSelect2('#nama_instansi', "{{ route('admin.instansis') }}", 'Pilih Instansi')
	
			$('#addDataModal').modal('show')
		})
```
### show btn modal edit
```bash
        $(document).on('click', '.btn-edit', function () {
			const userId = $(this).data('id')
	
			axios.get(`/admin/user/${userId}/edit`)
				.then(response => {
					const user = response.data
	
					$('#addDataModalLabel').text('Edit User')
					$('#addDataForm').data('id', user.id)
					$('#name').val(user.name)
					$('#email').val(user.email)
	
					if (!$('#level').hasClass('select2-hidden-accessible')) {
						initializeSelect2('#level', "{{ route('admin.levels') }}", 'Pilih Level');
						initializeSelect2('#nama_instansi', "{{ route('admin.instansis') }}", 'Pilih Instansi');
					}
	
					if ($('#level').find("option[value='" + user.level + "']").length) {
						$('#level').val(user.level).trigger('change')
					} else {
						const newOption = new Option(user.level_text, user.level, true, true)
						$('#level').append(newOption).trigger('change')
					}
	
					if ($('#nama_instansi').find("option[value='" + user.instansi_id + "']").length) {
						$('#nama_instansi').val(user.instansi_id).trigger('change')
					} else {
						const newOption = new Option(user.instansi_name, user.instansi_id, true, true);
						$('#nama_instansi').append(newOption).trigger('change')
					}
	
					$('#addDataModal').modal('show')
				})
				.catch(error => {
					showToast('error', 'Gagal mengambil data user.')
				});
		})
```
### Save to Database
```bash
        $('#saveData').on('click', function () {
			let saveButton = $(this)
			saveButton.prop('disabled', true).text('Proses...')
	
			let formData = {
				name: $('#name').val(),
				email: $('#email').val(),
				level: $('#level').val(),
				nama_instansi: $('#nama_instansi').val()
			}
	
			const password = $('#password').val()
			if (password) {
				formData.password = password
			}
	
			const userId = $('#addDataForm').data('id')
			const url = userId ? `/admin/user/${userId}` : "{{ route('admin.user.store') }}"
			const method = userId ? 'put' : 'post'
	
			axios({
				method: method,
				url: url,
				data: formData
			})
			.then(response => {
				showToast('success', response.data.message)
	
				$('#addDataForm')[0].reset()
				$('#addDataForm').removeData('id')
				$('#addDataModal').modal('hide')
				table.ajax.reload(null, false)
			})
			.catch(error => {
				if (error.response && error.response.status === 422) {
					const errors = error.response.data.errors
	
					$('.invalid-feedback').hide()
					$('.form-control').removeClass('is-invalid')
	
					Object.keys(errors).forEach(field => {
						$(`#${field}-error`).text(errors[field][0]).show()
						$(`#${field}`).addClass('is-invalid')
					});
				} else {
					showToast('error', 'Gagal menyimpan data. Silakan coba lagi.')
				}
			})
			.finally(() => {
				saveButton.prop('disabled', false).text('Simpan')
			});
		})
```
### Button Delete Confirmation with sweetalert2
```bash
        $(document).on('click', '.btn-delete', function () {
			const userId = $(this).data('id')
	
			Swal.fire({
				title: 'Apakah Anda yakin?',
				text: "User ini akan dihapus secara permanen!",
				icon: 'warning',
				showCancelButton: true,
				confirmButtonColor: '#3085d6',
				cancelButtonColor: '#d33',
				confirmButtonText: 'Ya, hapus!',
				cancelButtonText: 'Batal'
			}).then((result) => {
				if (result.isConfirmed) {
					axios.delete(`/admin/user/${userId}`)
						.then(response => {
							Swal.fire(
								'Terhapus!',
								response.data.message, // ambil message dari backend
								'success'
							);
							table.ajax.reload(null, false);
						})
						.catch(error => {
							Swal.fire(
								'Gagal!',
								'Terjadi kesalahan saat menghapus user.',
								'error'
							)
						})
				}
			})
		})
```
