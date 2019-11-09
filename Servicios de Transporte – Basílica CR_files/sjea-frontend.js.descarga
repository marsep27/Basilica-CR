(function( $ ) {
	$(document).ready(function() {
		var $form = $( '.sjea-elementor-form' );

		if ( $form.length > 0 ) {

			$form.each(function( i ) {
				var $this = $(this);
				var fields = $this.find('.elementor-field'); 

				fields.each(function( i ) {
					var $field = $(this);
					var map_value = $field.attr('data-map-value');

					$field.attr('name', 'param['+ map_value +']' );

				});

				$this.on( 'submit', function( event ) {
					event.preventDefault();

					var $submitButton = $this.find( '[type="submit"]' );

					if ( $this.hasClass( 'sjea-el-form-waiting' ) ) {
						return false;
					}

					$this
						.animate( {
							opacity: '0.45'
						}, 500 )
						.addClass( 'sjea-el-form-waiting' );

					$submitButton
						.attr( 'disabled', 'disabled' )
						.find( '> span' )
						.prepend( '<span class="elementor-button-text elementor-form-spinner"><i class="fa fa-spinner fa-spin"></i>&nbsp;</span>' );

					$this
						.find( '.sjea-el-message' )
						.remove();

					$this
						.find( '.sjea-el-error' )
						.removeClass( 'sjea-el-error' );

					$this
						.find( 'div.sjea-el-field-group' )
						.removeClass( 'error' )
						.find( 'span.elementor-form-help-inline' )
						.remove()
						.end()
						.find( ':input' ).attr( 'aria-invalid', 'false' );

					// var formData = $this.serializeArray().reduce(function(obj, item) {
					//     obj[item.name] = item.value;
					//     return obj;
					// }, {});

					var formData = $this.serialize();


					//console.log( formData );
					// return;
					// var formData = new FormData( $this[ 0 ] );
					// formData.append( 'action', 'sjea_add_subscriber' );
					// formData.append( 'referrer', location.toString() );
					
					$.ajax( {
						url: sjea.ajaxurl,
						type: 'POST',
						dataType: 'json',
						data: formData,
						// processData: false,
						// contentType: false,
						success: function( response, status ) {
							$submitButton
								.removeAttr( 'disabled' )
								.find( '.elementor-form-spinner' )
								.remove();

							$this
								.animate( {
									opacity: '1'
								}, 100 )
								.removeClass( 'sjea-el-form-waiting' );

							if ( ! response.success ) {
								if ( response.data.fields ) {
									$.each( response.data.fields, function( key, title ) {
										$this
											.find( 'div.sjea-el-field-group' ).eq( key )
											.addClass( 'sjea-el-error' )
											.append( '<span class="sjea-el-message sjea-el-message-danger elementor-help-inline elementor-form-help-inline" role="alert">' + title + '</span>' )
											.find( ':input' ).attr( 'aria-invalid', 'true' );
									} );
								}
								$this.append( '<div class="sjea-el-message sjea-el-message-danger" role="alert">' + response.data.message + '</div>' );
							} else {
								$this.trigger( 'submit_success' );
								$this.trigger( 'reset' );

								if ( '' !== response.data.message ) {
									$this.append( '<div class="sjea-el-message sjea-el-message-success" role="alert">' + response.data.message + '</div>' );
								}
								if ( '' !== response.data.link ) {
									location.href = response.data.link;
								}
							}
						},

						error: function( xhr, desc ) {
							$this.append( '<div class="sjea-el-message sjea-el-message-danger" role="alert">' + desc + '</div>' );

							$submitButton
								.html( $submitButton.text() )
								.removeAttr( 'disabled' );

							$this
								.animate( {
									opacity: '1'
								}, 100 )
								.removeClass( 'sjea-el-form-waiting' );

							$this.trigger( 'error' );
						}
					} );
				} );
			});

		}
	// SJEaFrontend = {

	// 	init: function() {
	// 		/* Subscribe Form */
	// 		$('.sjea-subscribe-form').on('click', SJEaFrontend._submitSubscribeForm);
			
			
	// 	},

	// 	_submitSubscribeForm: function(event){
	// 		event.preventDefault()
	// 		console.log( $(this) );
	// 	},
	// }


		//SJEaFrontend.init();
	});

})( jQuery );



